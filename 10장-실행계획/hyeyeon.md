# 통계 정보

통계 정보는 MySQL의 실행 계획에 가장 큰 영향을 미친다.
통계 정보를 테이블 및 인덱스에 대한 통계 정보와 히스토그램으로 나누어 살펴보자.

## 테이블 및 인덱스 통계 정보

비용기반 최적화에서 중요한 것은 통계 정보이다.(엉뚱한 방향으로 쿼리가 실행될 수 도 있음)
쿼리의 실행계획을 수립 할 때 실제 테이블의 데이터를 일부 분석해서 통계 정보를 보완해서 사용

### MySQL 서버의 통계 정보

MySQL 5.5 버전 까지는 각 테이블의 통계 정보가 메모리에서만 관리되어서 서버 재시작시 모두 사라졌다.
이후의 버전에서는 각 테이블의 통계 정보를 mysql 데이터베이스의 innodb_index_stats 테이블과 innodb_index_stats 테이블로 관리할 수 있게되어 재시작 되어도 통계 정보가 유지된다.
특정 테이블의 통계 정보를 영구적으로 관리하고 싶지 않을 경우 테이블 생성 시 STATS_PERSISTENT를 0으로 설정하면 된다.
STATS_PERSISTENT를 설정하지 않으면 innodb_stats_persistent 시스템 변수의 값에 따라 결정한다. (ON이면 영구 저장, OFF면 영구 저장 X)

테이블에 있는 인덱스들의 통계 정보는 위와 같이 저장되어 있다.

stat_name=’n_diff_pfx%’: 인덱스가 가진 유니크한 값의 개수
stat_name=’n_leaf_pages’: 인덱스의 리프 노드 페이지 개수
stat_name=’size’: 인덱스 트리의 전체 페이지 개수

employees 테이블의 통계 정보는 위와 같이 저장되어 있다.

n_rows: 테이블의 전체 레코드 건수
clustered_index_size: 프라이머리 키의 크기(InnoDB 페이지 개수)
sum_of_other_index_sizes: 프라이머리 키를 제외한 인덱스의 크기(InnoDB 페이지 개수)
통계 정보는 아래와 같은 이벤트들이 발생하면 갱신된다.

테이블이 새로 오픈되는 경우
테이블의 레코드가 대량으로 변경되는 경우
ANALYZE TABLE 명령이 실행되는 경우
SHOW TABLE STATUS 명령이나 SHOW INDEX FROM 명령이 실행되는 경우 하지만 갑자기 통계 정보가 변경되면 의도치 않게 실행 계획이 변경되는 문제가 발생할 수 있는데, innodb_stats_auto_recalc 시스템 변수의 값을 OFF로 설정하면 이를 막을 수 있다.
또한, 통계 정보를 자동으로 수집할지 여부도 옵션을 통해 테이블 단위로 조정할 수 있다.
통계 정보를 수집할 때 몇 개의 InnoDB 테이블 블록을 샘플링할지 설정하는 시스템 변수가 2개있다.
innodb_stats_transient_sample_pages: 기본값=8, 자동으로 통계가 수집될 때 8개의 페이지만 분석하여 통계 정보로 활용함을 의미한다.
innodb_stats_persistent_sample_pages: 기본값=20, ANALYZE TABLE 명령이 실행되면 20개의 페이지만 분석하여 통계 정보로 활용함을 의미한다.
→ 정확도를 높히고 싶다면 위 시스템 변수 값을 올리면 되지만, 통계 수집 시간이 길어지므로 주의해야한다.

### 히스토그램

기존 통계 정보만으로는 최적의 실행 계획을 수립하기에는 많이 부족했다.

MySQL 8.0부터는 컬럼의 데이터 분포도를 참조할 수 있는 히스토그램을 활용할 수 있다.

히스토그램 정보는 컬럼 단위로 관리되는데, 이는 자동으로 수집되지 않고 ANALYZE TABLE … UPDATE HISTOGRAM 명령을 실행해 수동으로 수집된다.

이 히스토그램 정보를 조회하려면 column_statistics 테이블을 SELECT하면 된다.

히스토그램은 싱글톤 히스토그램, 높이 균형 히스토그램 두 가지가 지원된다.

### 코스트 모델

전체 쿼리의 비용을 계산하는데 필요한 단위 작업들의 비용을 코스트 모델이라고 한다.

MySQL의 코스트 모델은 다음 2개 테이블에 저장되어 있는 설정값을 사용한다.

server_cost: 인덱스를 찾고 레코드를 비교하고 임시 테이블 처리에 대한 비용 관리
engine_cost: 레코드를 가진 데이터 페이지를 가져오는데 필요한 비용 관리
두 테이블은 아래 5개의 컬럼을 공통으로 가지고있다.

cost_name: 코스트 모델의 각 단위 작업
default_value: 각 단위 작업의 비용(기본값, MySQL 서버 소스 코드에 설정된 값)
cost_value: DBMS 관리자가 설정한 값(NULL이면 default_value 값 사용)
last_updated: 단위 작업의 비용이 변경된 시점
comment: 비용에 대한 추가 설명
engine_cost 테이블은 아래 2개의 컬럼을 더 가지고 있다.

engine_name: 비용이 적용된 스토리지 엔진
device_type: 디스크 타입

| 테이블 이름 | cost_name                    | default_value | 설명                             |
| ----------- | ---------------------------- | ------------- | -------------------------------- |
| engine_cost | io_block_read_cost           | 1             | 디스크 데이터 페이지 읽기        |
| engine_cost | memory_block_read_cost       | 0.25          | 메모리 데이터 페이지 읽기        |
| server_cost | disk_temptable_cost          | 20            | 디스크 임시 테이블 생성          |
| server_cost | disk_temptable_row_cost      | 0.5           | 디스크 임시 테이블의 레코드 읽기 |
| server_cost | key_compare_cost             | 0.05          | 인덱스 키 비교                   |
| server_cost | memory_temptable_create_cost | 1             | 메모리 임시 테이블 생성          |
| server_cost | memory_temptable_row_cost    | 0.1           | 메모리 임시 테이블의 레코드 읽기 |
| server_cost | row_evaluate_cost            | 0.1           | 레코드 비교                      |

코스트 모델은 각 단위 작업에 설정되는 비용이 커지면 어떤 실행 계획의 비용이 변하는지 파악하는 것이 중요하다.
웬만하면 위 테이블들의 default_value를 바꾸지 말자

# 실행 계획 확인

MySQL의 실행 계획은 DESC 또는 EXPLAIN 명령으로 확인할 수 있다.

실행 계획의 포맷은 아래와 같이 테이블, 트리, JSON 3가지 중 하나를 선택할 수 있다.

EXPLAIN [FORMAT=TREE or JSON] (테이블이 기본값)

## 실행 계획 출력 포맷

## 쿼리의 실행 시간 확인
