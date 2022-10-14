---
title: "주니어 분석가로 살아가면서 얻은 PostgreSQL 관련 소소한 정보들"
categories:
  - 주니어 데이터 분석가
tags:
  - postgres_fdw
  - pg_cron
  - PostgreSQL
---
     
데이터 분석가는 회사에 입사하면 먼저 그 회사의 데이터베이스들을 접하게 됩니다. 데이터베이스에는 회사의 여러 데이터들이 담겨 있고, 분석가는 그 데이터를 입맛에 맞게 전처리와 가공 과정을 거치며 '분석할 수 있는 상태'의 데이터로 만듭니다. 하지만 항상 데이터베이스가 *분석가가 분석하기 편한 상태*로 존재하지는 않습니다. 저는 이번에 제가 겪은 *분석하기 불편한 상태*의 데이터를 가지고 분석하기 위해 어떻게 했는지에 대한 글을 써보려고 합니다. 누군가는 이미 알고 있는 정보일 수도 있지만, 모르는 사람들에게는 소소한 팁 정도는 되리라 믿고 글을 써내려가보고자 합니다.   
*해당 글은 PostgreSQL을 기반으로 작성되었습니다.     
     
     
---
     
     
# 외부 DB의 테이블과 로컬 DB의 테이블을 조인시키고 싶어요   
회사에는 여러 개의 데이터베이스가 존재합니다. 서비스가 여러 개라서 DB 또한 여러 개일수도 있고, 아니면 목적에 맞게 DB를 설계하기 때문에 여러 개일 수도 있습니다. 분석가 입장에서는 하나의 데이터베이스에 원하는 모든 데이터가 다 들어있어서 굳이 타 데이터베이스까지 엮어볼 필요가 없는 경우가 제일 편하겠지만, DB는 분석가를 생각하며 설계되지는 않습니다. 저 또한 A라는 데이터와 B라는 데이터를 엮어서 살펴 볼 일이 있었는데, 안타깝게도 데이터A와 데이터B는 각기 다른 DB 테이블들에 존재하는 상태였습니다. 이 때 저는 **postgres_fdw**를 활용했습니다. postgres_fdw는 dblink 같이 물리적으로 떨어져 있는 원격 테이블에 접속하기 위해 사용되는 기능입니다. 외부 DB data 실시간 연계로, 속도 차이가 없으며 로컬 DB와 join 쿼리도 가능합니다.     
     
![postgre fdw](https://user-images.githubusercontent.com/104043279/185560971-1fe52f39-bbb2-4d7b-8772-9ca1785f6890.png)    
     
* 장점:    
superuser 권한이 없어도 가능하다.    
세션 단위인 dblink와 달리 접속 설정이 지속된다.     
외부 DB 실시간 데이터 조회 가능하다.     
로컬 DB와 join 가능하다.     
스키마 단위 그대로 가져올 수 있다.     
     
* 단점:     
조회하려는 외부 DB 테이블과 동일한 이름의 테이블이 존재하면 존재하는 테이블을 삭제하거나 이름을 다르게 테이블을 생성하고 컬럼 타입을 대조해서 하나하나 생성해야한다.     
    
설치 방법은 아래와 같습니다.    


```
-- CREATE EXTENSION
CREATE EXTENSION IF NOT EXISTS postgres_fdw;

-- CREATE SERVER
CREATE SERVER [server_name] FOREIGN DATA WRAPPER postgres_fdw 
OPTIONS (host '외부DB 주소', port '외부DB 포트', dbname '외부DB DB명');

-- CREATE USER MAPPING
CREATE USER MAPPING FOR postgres SERVER [server_name] 
OPTIONS (user '외부DB 유저명', password '외부DB 패스워드')
```

위와 같은 단계를 모두 완료했다면 이제는 로컬DB에 외부DB의 테이블들을 불러올 수 있게 되었습니다.   


```
-- CREATE FOREIGN TABLE   
CREATE FOREIGN TABLE [로컬DB에서 사용할 테이블명] (
    name text,
	id text,
	...
)
SERVER [server name]
OPTIONS (schema_name '외부DB schema', table_name '외부DB 테이블명');
```
   
(  ) 안에 들어갈 내용은 가져오고자 하는 원격DB 테이블의 DDL을 보면 됩니다. **DDL에서 CONSTRAINT 부분을 제외하고 가져오면 된다.*    
이렇게 하면 로컬DB에 외부DB의 테이블이 우리가 설정한 테이블명대로 생겨난 것을 볼 수 있습니다. 해당 테이블과 기존 로컬DB의 테이블을 조인시키는 것 또한 가능해졌습니다. 주의해야 할 점은 테이블을 새롭게 생성했다는 개념보다는, 복사해 붙여넣었다는 개념에 더 가깝다는 점입니다. 외부DB에서 해당 테이블이 삭제되거나 제거될 경우 로컬DB에서도 더이상 그 테이블을 볼 수 없게 됩니다.   
   
     
---
    
     
# 일정 주기마다 특정 쿼리의 결과를 저장해 놓아야 해요  
데이터 분석가로써 회사 업무를 진행하다보면 적어도 한 번은 매일, 매주, 매달 등 일정 주기마다 체크해야 할 데이터를 만나게 됩니다. 특히나 체크하는 데서 그치지 않고, 쿼리 결과 데이터를 일정 주기를 기준으로 쌓아놔야 한다고 하면, 쿼리를 돌리고 그 결과를 특정 테이블에 insert하는 작업을 주기적으로 반복해야 합니다. 한 번의 쿼리를 돌리는 것 자체가 그렇게 어렵고 공수가 큰 일은 아니지만, 그걸 일정 주기마다 계속해서 내가 해줘야한다고 생각하면 부담스럽고 귀찮게 느껴집니다. 주말이나 휴일에도 쿼리를 돌리기 위해서 노트북을 켤 수도 없는 일이고요. 이럴 때 유용하게 쓰일 수 있는 것이 바로 **pg_cron**입니다.   
      
![표](https://user-images.githubusercontent.com/104043279/185561303-62099c25-4c1a-455b-8c1f-38c699ce09e8.png)
      
pg_cron은 데이터베이스 내부에서 확장으로 실행되는 PostgreSQL용 간단한 cron 기반 작업 스케줄러입니다. 일반 cron과 동일한 구문을 사용하지만 데이터베이스에서 직접 PostgreSQL 명령을 예약할 수 있습니다. 이 pg_cron을 이용해 주기적으로 쿼리 결과를 스냅샷을 뜰 수 있습니다.  
   
## pg_cron 설치
pg_cron을 설치하는 방법에는 3가지가 있습니다.      


```
# 1. PGDG 를 사용하여 PostgreSQL가 설치된 Red Hat, CentOS, Fedora, Amazon Linux에 설치
# Install the pg_cron extension
sudo yum install -y pg_cron_12

# 2. apt.postgresql.org 를 사용하여 PostgreSQL 12가 설치된 Ubuntu, Debian에 설치
# Install the pg_cron extension
sudo apt-get -y install postgresql-12-cron

# 3. 소스에서 빌드하여 pg_cron을 설치
git clone https://github.com/citusdata/pg_cron.git
cd pg_cron
# Ensure pg_config is in your path, e.g.
export PATH=/usr/pgsql-12/bin:$PATH
make && sudo PATH=$PATH make install
```
     
      
## pg_cron 설정   
설치가 끝났으면 PostgreSQL DB 인스턴스와 연결된 파라미터 그룹을 수정하고 pg_cron을 shared_preload_libraries 파라미터 값에 추가합니다. 이 변경 사항을 적용하려면 PostgreSQL DB 인스턴스를 다시 시작해야 합니다.     
PostgreSQL DB 인스턴스가 다시 시작된 후 rds_superuser 권한이 있는 계정을 사용하여 다음 명령을 수행합니다. 예를 들어 RDS for PostgreSQL DB 인스턴스를 생성할 때 기본 설정을 사용한 경우 사용자 postgres로 연결하고 확장을 생성합니다.   


```
CREATE EXTENSION pg_cron;
```
pg_cron 스케줄러는 postgres라는 기본 PostgreSQL 데이터베이스에 설정됩니다. pg_cron 객체가 이 postgres 데이터베이스에 생성되고 모든 예약 작업이 이 데이터베이스에서 실행됩니다.     
     
     
이제 주기적으로 돌아갔으면 하는 쿼리를 크론 스케쥴로 등록하면 됩니다. 크론 스케줄을 하기에 앞서, 로컬DB에 스냅샷한 결과값이 들어갈 테이블을 먼저 만들어주어야 합니다.   


```
select cron.schedule(
    '테이블에 대한 설명(테이블명과 동일하게 해도 상관x)',
    '* * * * *',
    $$
    insert into [테이블명]
    (
        SELECT
            date(current_timestamp  at time zone 'Asia/Seoul' - '1day'::interval) snap_date
            , column1, column2
        from
            TABLE
    )
    $$
);
```

'* * * * *'은 스케줄링이 언제 돌아갈지 주기를 설정할 수 있는 부분입니다. 각 순서마다 뜻하는 시간기준이 다릅니다.      


```
 ┌───────────── min (0 - 59)
 │ ┌────────────── hour (0 - 23)
 │ │ ┌─────────────── day of month (1 - 31)
 │ │ │ ┌──────────────── month (1 - 12)
 │ │ │ │ ┌───────────────── day of week (0 - 6) (0 to 6 are Sunday to
 │ │ │ │ │                  Saturday, or use names; 7 is also Sunday)
 │ │ │ │ │
 │ │ │ │ │
 * * * * *
```

*는 와일드카드로 "매 기간마다 실행"을 의미하고 특정 숫자는 "오직 이 시간에만"을 의미합니다.   
하지만 이 시간조건은 한국 시간을 기준으로 설정되어 있지 않기 때문에, 한국 시간을 기준으로 설정하려면 +15시간을 해야합니다.   
> **ex:** 매일 한국 시간 기준 10시 30분에 스케줄링이 돌아가게끔 하고 싶으면   
'30 ' || (10 + 15) % 24 || ' * * *'   
라고 해야 합니다.   
   
언제 스냅샷이 생성되었는지 알아보기 위한 시간 컬럼을 생성할 때도 반드시 Asia/Seoul 시간문을 써주어야 합니다.(where절 등 조건 걸 때도 마찬가지)   
> **ex:** now() at time zone 'Asia/Seoul' as date   
date(now() at time zone 'Asia/Seoul' - '1day'::interval) date   
date(current_timestamp at time zone 'Asia/Seoul' - '1day'::interval) date    
     
스케줄이 제대로 등록되었는지를 보려면 다음과 같이 확인합니다.   

```
select * from cron.job;  
```
     
스케줄링이 실행된 기록을 보려면 다음과 같이 확인합니다.   

```
select * from cron.job_run_details;
```
     
스케줄이 건 시간 주기를 바꾸고 싶다면 다음과 같이 쿼리를 작성하고 돌리면 됩니다.   

```
update cron.job set schedule = '수정할 시간 주기' where jobid = [수정하고자 하는 스케줄쿼리 jobid];
``` 
    
     
---
     
    
아직 저도 많이 배우고 성장해야 할 입장이라, 제가 썼던 방식이 최고의 대안은 아닐 수도 있습니다. 제가 알아내지 못했을뿐, 더 효율적이고 나은 선택지가 있을 수도 있습니다. 다만 저는 '가장 효율적이지는 않더라도 주어진 일은 어떻게든 처리되게 하자'라는 마음으로 업무를 진행했고, 그런 측면에서 저에게는 최선의 방식이었습니다. 이 글을 읽는 사람들 중 한 명이라도 현재 업무 진행이 막혀있는데, 이러한 방식으로 진행하면 일을 처리할 수 있겠구나 하는 사람이 있다면 정말 기쁠 것 같습니다.    
     
     
---
     
     
*참조:    
<<<<<<< HEAD
https://www.postgresql.fastware.com/postgresql-insider-fdw-ove   
https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/UserGuide/PostgreSQL_pg_cron.html   
https://aws.amazon.com/ko/blogs/database/schedule-jobs-with-pg_cron-on-your-amazon-rds-for-postgresql-or-amazon-aurora-for-postgresql-databases/    
https://github.com/citusdata/pg_cron   
=======
[https://www.postgresql.fastware.com/postgresql-insider-fdw-ove](https://www.postgresql.fastware.com/postgresql-insider-fdw-ove)   
[https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/UserGuide/PostgreSQL_pg_cron.html](https://docs.aws.amazon.com/ko_kr/AmazonRDS/latest/UserGuide/PostgreSQL_pg_cron.html)   
[https://aws.amazon.com/ko/blogs/database/schedule-jobs-with-pg_cron-on-your-amazon-rds-for-postgresql-or-amazon-aurora-for-postgresql-databases/](https://aws.amazon.com/ko/blogs/database/schedule-jobs-with-pg_cron-on-your-amazon-rds-for-postgresql-or-amazon-aurora-for-postgresql-databases/)    
[https://github.com/citusdata/pg_cron](https://github.com/citusdata/pg_cron)   
>>>>>>> eef26f63b3c998b01a35b181e48608a89ba01770
