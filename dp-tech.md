# DP 팀에서 사용하는 기술들

2021-03-23 기준으로 중요도가 높은 기술을 순서대로 정리해보았습니다.
요즘 DP팀 메인 이슈는 Jenkins를 Batch Job을 관리하는 Tool인 Airflow로 옮기는
작업입니다.

# 1. [Airflow](http://airflow.apache.org/)

2021-03-23 기준 Airflow Version은 2.0.1입니다.

## 1.1 Airflow만의 강점

1. Scalable:
  * 모듈식 아키텍쳐(Modular Architecture)
  * 메시지 큐를 사용하여 임의 수의 작업자를 조정함.
    * (producer-consumer design pattern 참고)
  * 확장이 용이하다.
2. Dynamic: 동적 파이프라인 생성이 가능하다.
3. Extensible: 사용자 환경에 맞는 추상화 수준에 맞게 라이브러리를 확장할 수 있다.

## 1.2 Features

* Python 사용함
* 타사 서비스(GCP, AWS, MA)에 쉽게 적용이 됨
* UI 괜찮아서 사용하기 괜찮은거 같음
* Open source

## 1.3 Concepts

Workflow를 describe, execute, monitor 하는 플랫폼.

![Image of Airflow Architecture](https://airflow.apache.org/docs/apache-airflow/stable/_images/arch-diag-basic.png)

### Glossary 

* Webserver, Scheduler, Worker
  - Webserver, Scheduler: 로컬 컴퓨터와 DB 간의 상호작용을하거나 웹서버와 스케쥴러가 각각의 프로세스를 실행하기도 한다.
      - 일반적으로는 스케쥴러가 DAG Run을 만듦. (외부 트리거에 의해 생성되기도.)
  - Worker: Airflow 아키텍처와 메타데이터 저장소의 구성 요소와 상호 작용하는 별도의 프로세스

* DAG (Directed Acyclic Graph)
  - Python으로 작성된 작업(work 또는 task) 또는 작업이 수행되어야 하는 순서(dependencies)
  - Airflow에서 실행하려는 데이터 파이프라인을 나타내는 작업 모음. 해당 작업의 관계와 종속성을 반영하는 방식으로 구성된다.
  - DAG 파일 위치는 Airflow 구성 파일에 지정되지만 웹 서버, 스케줄러 및 작업자가 액세스할 수 있어야 함.
  - DAG는 Airflow 구성 작업(Constituent Task)이 하는 일에 관심 X.
  - 어떤 일이 적시에 일어나는지, 올바른 순서로 일어나는지, 또는 예기치 않은 문제에 대한 올바른 처리를 하는지를 본다. 

* DAG Run
  - 특정 논리 날짜와 시간에 대한 DAG 인스턴스

* Operator
  - 일부 작업을 수행하기 위한 템플릿 역할을 하는 클래스

* Task
  - Python으로 작성된 operator를 구현함으로써 작업을 정의한다.

* Task Instance
  - DAG에 할당된 특정 DAG 실행과 관련된 상태를 지닌 작업 인스턴스

* execution_date 
  - DAG 실행과 해당 태스크 인스턴스에 대한 논리적 날짜와 시간

* Metadata DB
  - SQL 데이터베이스를 사용하여 실행 중인 데이터 파이프라인에 대한 메타데이터를 저장.
  - 다이어그램에서는 Postgres로 표시했음을 참고하자. MySQL 또한 지원된다.

* `airflow.cfg`
  - 웹서버, 스케쥴러, 워커가 접근하는 Airflow 설정파일
  - (Dable) airflow 서버 생성하는 것을 자동화 시키는 업무를 할 것 같음.
  - (Dable) S3에 대한 로그를 만들 것 같음.

* workflow
  - DAGs와 Operators를 결합 -> TaskInstances를 생성 -> 복잡한 workflow 빌드

## 1.4 Core Ideas: DAGs

1. Global로 정의해야 한다. local에서 정의한 것은 local에서만 보임. Scope 조심하자!
2. Default Argument가 있기 때문에 common parameter를 여러 번 입력할 필요 없이 쉽게 적용할 수 있다.
3. DAG를 context manager로 사용하면 해당 DAG에 새 연산자를 자동으로 할당할 수 있다.
4. Authoring DAG의 새로운 스타일로 TaskFlow API를 사용할 수도 있다.
5. context manager를 사용해서 DAG를 생성하면 `@` decorator를 사용해서 DAG를 생성하는 함수를 만들 수도 있음

## 1.5 DAG Runs

* 특정 `execution_date`(DAG run과 해당 태스크 인스턴스가 실행하는 논리적 날짜, 시간)에 대해 실행되는 태스크 인스턴스를 포함하는 DAG를 인스턴스화 한 것.
  * `execution_date`로 태스크 인스턴스는 원하는 논리 날짜 및 시간에 대한 데이터를 처리
  * 주의사항: task_instance 또는 DAG 실행의 실제 시작 날짜가 현재일 수 있지만 논리 날짜는 3개월 전일 수 있음.

* DAG 실행과 해당 실행 내에 생성된 모든 태스크 인스턴스가 동일한 execution_date로 인스턴스화되므로 논리적으로 DAG 실행이 모든 태스크를 실행 중인 이전 날짜 및 시간에 시뮬레이션하는 것으로 생각할 수 있음.
  * DAG : DAG Run = 1 : N 일 수도 있다. 

## 1.6 Tasks

* DAG에서 node에 해당되고 work 집합을 의미한다.
* 각각의 task는 Operator를 구현한다.

1.6.1 Tasks간의 관계

* DAG에서 각 task는 노드다. 단 task_1 -> task_2 로의 종속성 존재.
  * `task_1 >> task_2` # Define dependencies
  * task_1는 task_2의 upstream
  * task_2는 task_1의 downstream
  * task_2는 task_1이 성공적으로 완료될 때까지 기다렸다가 시작한다.

1.6.2 `@task` : Python task decorator

* `@task`는 모든 python 함수를 airflow 연산자로 변환
* decorated function을 호출해서 연산자를 실행에 필요한 arguments, key arguments를
설정한다.
  * decorated function은 두 번 이상 호출할 수 있다.
  * decorated function은 생성된 각 연산자에 대한 고유한 task_id를 자동으로
      생성할 수 있다.
  * task_id를 명시적으로 주지 않은 태스크가 많으면 다음과 같은 문제가 발생한다.
    * task__n 이전에 새 태스크를 삽입하면 이후 task_id는 하나씩 뒤로 밀림.
    * 시간이 지남에 따라 변경된 DAG 기록 로그나 DAG Runs에 혼동이 생김. 

1.6.3 current context로 접근하기

* current execution context를 검색하려면 `get_current_context` 메서드를 사용.
* `@task` decorator 사용시 유용하다.
* current context는 작업 실행 중에만 접근할 수 있다.
  * `pre_execute` 또는 `post_execute` 중에는 접근 불가능함.

## 1.7 Task Instances

1.7.1 특징

* 태스크의 특정 실행을 나타냄. DAG 실행에 속함.
  * 태스크는 DAG에서 정의됨.
  * 인스턴스화됨. 실행 가능한 엔티티를 가짐.
* DAG, 태스크 및 지정 시점(`execution_date`)의 조합
* 표시 상태: "running", "success", "failed", "skipped", "up for retry"

1.7.2 Task Instances 간의 관계

task_1은 task_2의 upstream이지만 `execution_date`는 서로 같음.

## 1.8 Task Lifecycle

Task는 start부터 completion까지 다양한 스테이지를 나타냄.
Airflow UI에서 각 스테이지는 각기 다른 색깔로 보여짐.

* 가장 이상적인 플로우

1. No status (스케쥴러가 빈 태스크 인스턴스를 만든 상태)
2. Scheduled (스케쥴러가 태스크 인스턴스를 실행하려고 결정한 상태)
3. Queued (스케쥴러가 executor에 태스크를 보내서 큐를 실행하려고 하는 상태)
4. Running (워커가 태스크를 선택하고 태스크가 돌아가는 상태)
5. Success (태스크가 완료된 상태)

## 1.9 [Operators](https://airflow.apache.org/docs/apache-airflow/stable/concepts.html#operators)

문서 참고할 것

1.9.1 Sensors

1.9.2 DAG Assignment

1.9.3 Bitshift Composition

1.9.4 Relationship Builders

## 1.10 Best Practices

    DAG 생성

    * DAG 객체를 생성하는 파이썬 코드 작성하기
    * Expectation을 만족하는지 코드 테스트하기

### 1.10.1 Writing a DAG

1.10.1.1 태스크 생성하기

* Airflow에서 태스크 다루는 일은 DB 트랜잭션과 비슷하다고 생각하면 된다.
* Task에 불완전한 결과를 생성하면 안된다.
* Task를 재실행할 때마다 동일한 결과를 도출해야 함.
  * Task re-run 동안 INSERT 사용 X. INSERT 문이 DB에 중복된 행으로 이어지기
      때문. 
  -> UPSERT 사용할 것
* 특정 파티션에서 읽고 쓴다.
  * 절대로 Task에서 사용가능한 latest data를 읽지 말기. 
    * 재실행 사이에 다른 사람이 입력 데이터를 업데이트 해서 다른 결과가
        도출되기 때문. 
    -> 특정 파티션의 입력 데이터를 읽는 것이 좋음.
  * `execution_date`를 특정 파티션으로 사용할 수 있다.

1.10.1.2 태스크 삭제하기

* DAG의 태스크 절대로 삭제하지 말 것.
  * 삭제시 Airflow UI에서 historical 태스크 정보가 사라진다.
  * 삭제할 필요가 있더라도 DAG를 새로 만드는 것을 추천함.

1.10.1.3 커뮤니케이션

* Airflow는 Kubernetes executor를 사용하는 경우, 다른 서버들에서 DAG 작업을 실행함
* 따라서 설정 파일 등을 로컬 파일시스템에 저장해선 안됨.
  * 다음 태스크는 액세스하지 않고 다른 서버에서 실행될 수 있기 때문임. 
* 로컬 executor의 경우에는 파일을 디스크에 저장하면 재시도하기가 더 어려워질 수 있음.
  * DAG에 있는 다른 태스크가 삭제한 구성 파일이 필요할 수 있기 때문임.
* 가능하면 XCom을 사용해서 작업 간에 작은 메시지를 주고받자.
  * 더 큰 데이터를 Task에 전달하는 좋은 방법은 S3/HDFS와 같은 원격 저장소를 사용하는 것.
  * 예를 들어 S3 데이터를 저장하는 태스크가 있는 경우,
    * 해당 태스크는 Xcom의 출력 데이터에 대한 S3 경로를 푸시할 수 있음. 
    * 다운스트림 태스크는 XCom에서 경로를 가져와 데이터를 읽는 데 사용할 수 있음.
* 암호 또는 토큰과 같은 인증 매개 변수를 태스크 내부에 저장해서는 안됨.
  * 가능하면 `Connections`을 사용
  * Airflow 백엔드에 데이터를 안전하게 저장해서 고유한 connection ID를 사용하여 데이터를 검색하자.

1.10.1.4 변수

* 변수는 Flow의 메타데이터 DB에 대한 연결을 만들어 값을 가져오므로 가능한 경우 연산자의 `execute()` 메서드 외부에 있는 변수나 Jinja 템플릿은 사용하지 않아야 한다. 사용하게 되면 구문 분석 속도가 느려지고 DB에 추가 로드가 발생할 수 있음.

* Airflow는 특정 기간에 백그라운드에 있는 모든 DAG를 파싱한다. 
기본 기간은 processor_poll_interval 구성을 사용하여 설정되며, 기본값은 1초. 
Airflow는 구문 분석 중에 각 DAG에 대한 메타데이터 DB에 대한 새 연결을 생성하며 이로 인해 연결이 많이 생길 수 있음.

* 변수를 사용하는 가장 좋은 방법은 Jinja 템플릿을 사용하는 것. 
  * 작업이 실행될 때까지 값 읽기를 지연시킵니다.
  * `{{ var.value.<variable_name> }}`나 
  * 또는 `{{ var.json.<variable_name> }}` 문법을 사용하면 된다.

### 1.10.2 Testing a DAG

1.10.2.1 DAG Loader Test

* 기본 컨셉 : DAG에 로드할 때 오류가 발생하는 코드가 없는지 확인해야 함.
  * `python your-dag-file.py` 명령어 실행했을 때 아무 에러가 발생하지 않으면 DAG에 에러가 없다는 것을 확인할 수 있다.

* 어떤 것을 테스트 하는가?
  * Testing은 스크립트를 실행했을 때 파이프라인이 잘 파싱되는지 확인해볼 수 있음.
  * 컨텍스트 안에 특정된 `execution_date`에 대한 task instance를 실행해서 테스트 해볼 수 있음.
  * 다른 것들이 통과되면 `backfill`을 실행해 볼 것.
  
        * backfill
        - 의존성을 존중하고 로그를 파일로 내보낸다.
        - 상태를 기록하기 위해서 DB와 커뮤니케이션 함.
        - 웹 서버가 설치되어 있다면 진행 상황을 추적할 수 있음.

1.10.2.2 Unit Tests

custom operator 만들어서 유닛테스트 할 수도 있음.

1.10.2.3 Self-Checks

DAG에서 검사를 구현하여 작업이 예상대로 결과를 생성하는지 확인할 수 있음.
예를 들어 파티션이 S3에 생성되었는지 확인할 수 있음. 또는 데이터가 올바른지 확인할 수 있음.

1.10.2.4 Staging Environment

* 가능한 프로덕션에서 배포하기 전에 전체 DAG run을 테스트하기 위한 준비 환경을 유지할 것. 
* DAG가 S3 작업의 출력 경로 또는 구성을 읽는 데 사용되는 데이터베이스 등 변수를 변경하도록 매개 변수화되었는지 확인할 것. 
* DAG 내부에서 값을 하드코딩하지 말고 환경에 따라 수동으로 변경할 것.

### 1.10.3 Mocking variables and connections

* 테스트 실행 속도를 높이기 위해서 mock 개체를 DB에 저장하지 않고도 시뮬레이션 해야 함.
* `unitest.mock.patch.dict()`를 사용할 것.