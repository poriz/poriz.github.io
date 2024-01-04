---
layout: post
title: 데브코스 53일차 - Jinja, Dag Dependencies
sitemap: false
categorys:
  - dataengineering
  - camp
---

# 데브코스 53일차 - Jinja, Dag Dependencies

## Jinja Template

: python에서 널리 사용되는 템플릿 엔진이다. Django 템플릿 엔진에서 영감을 받아 개발되었다고 한다. 현재는 Flask에서 많이 사용된다고 한다.

- 변수를 이중 중괄호로 감싸 사용 `<h1>name: {% raw %}{{name}}{% endraw %}<h1>`
- 제어문은 퍼센트 기호를 사용한다.

```python
# 반복문 예시
{% raw %} {% for item in items %} {% endraw %}
	...
{% raw %} {% endfor %} {% endraw %}
```


### Jinja + Airflow

: 작업 이름, 파라미터 또는 SQL 쿼리와같은 작업 매개변수를 템플릿화된 문자열로 정의 할 수 있다.

- execution_date를 코드 내에서 쉽게 사용 가능하다. : {% raw %}`{{ ds }}`{% endraw %}
- BashOperator에서의 사용 방식 → bash_command에서 jinja사용 가능

```python
# params를 활용해서 사용도 가능하다.
task = BashOperator(
	...
	bash_command='echo "test,{% raw %} {{params.name}}{% endraw %}"",
	params={'name':'pori'},
	dag=dag
)
```

- 각 Operator에서 어떤 Parameters에 jinja가 사용가능한가?
: docs를 확인해서 다음과 같이 Parameters 뒤에 (templated)가 적혀있는 경우에 사용가능하다.

![](https://velog.velcdn.com/images/pori/post/090ad327-dde9-4a18-b9b5-4ef690dc9c8b/image.png)


[Operators — Airflow Documentation](https://airflow.apache.org/docs/apache-airflow/stable/core-concepts/operators.html)

- Airflow에서 사용가능한 jinja들 : https://airflow.apache.org/docs/apache-airflow/stable/templates-ref.html


| {% raw %} {{ ds }} {% endraw %} | 연도-월-일 |
| --- | --- |
| {% raw %} {{ ds_nodash }} {% endraw %} | 대시없이 ds 출력 |
| {% raw %} {{ ts }} {% endraw %} | 연도-월-일-시-분-초 |
| {% raw %} {{ dag }} {% endraw %} | dag이름 |
| {% raw %} {{ task }} {% endraw %} | task에 대한 정보 |
| {% raw %} {{ dag_run }} {% endraw %} |  |
| {% raw %} {{ var.value}}: {{ var.value.get(’my.var’, ‘fallback’) }} {% endraw %} | Variable 읽어오기 (value) |
| {% raw %} {{ var.json }}: {{ var.json.my_dict_var.key1 }} {% endraw %} | Variable 읽어오기 (json) |
| {% raw %} {{ conn }}: {{ conn.my_conn_id.login }}, {{ conn.my_conn_id.password }} {% endraw %} | Connection 생성 |


---

## Dag를 실행하는 방법

- 주기적 실행 방법: schedule을 지정해서 수행하기
- 다른 Dag에 의해 트리거하기
    - Explicit Trigger: DAG A가 명시적으로 DAG B를 트리거 → TriggerDagOperator
    - Reactive Trigger: B가 A가 끝나기를 대기, A는 이를 알지 못한다. → ExternalTaskSensor

### TriggerDagRunOperator

- jinja 사용가능 params : trigger_dag_id, conf, execution_date
- conf : 다음 DAG에 넘기고 싶은 정보. `conf = { ‘name’: ‘pori’ }`
    - 다음 DAG에서 jinja로 접근하기: `{{ dag_run.conf[”name”] }}`
    - PythonOperator에서 접근: `kwargs['dag_run'].conf.get('name')`
- 참고: airflow.cfg의 `dag_run_conf_overrides_params`가 True로 되어야한다.

```python
# TriggerDagRunOperator
trigger_task = TriggerDagRunOperator(
	...
	conf = {...}
	execution_date = {% raw %} '{{ ds }}' {% endraw %}
	...
	dag=dag
)
# targetDAG
task1 = BashOperator(
	...
	bash_command ={% raw %} """echo '{{ds}}, {{ dag_run.conf.get("name","none")}}' """ {% endraw %}
)
```

### Sensor

: 특정 조건이 충족될 때까지 대기하는 Operator, 외부 리소스의 가용성이나 특정 조건의 완료와 같은 상황 동기화에 유용하게 사용된다.

- 내장 Sensor
    - FileSensor: 지정된 위치에 파일이 생길 때까지 대기
    - HttpSensor: HTTP 요청을 수행하고 지정된 응답이 대기
    - SqlSensor: SQL DB에서 특정 조건을 충족할 때까지 대기
    - TimeSensor: 특정 시간에 도달할 때까지 워크플로를 일시중지
    - ExternalTaskSensor: 다른 Airflow DAG의 특정 작업 완료를 대기
- 주기적인 poke : mode를 사용해서 방법을 선택한다.
    - poke: 주기적으로 체크하기, 하나의 worker에서 전담해서 체크 → 체크 주기가 명확해진다.
    - reschedule: worker를 릴리스하고, 다시 잡아서 poke수행 → 상황에 따라서 worker를 잡는것이 힘들 수 있다.

### ExternalTaskSensor

: 앞의 DAG의 특정 Task가 완료되었는지를 확인한다.

- execution date, schedule interval이 동일해야한다.
- 서로 다른 interval을 갖는 경우에는 execution_date_fn을 사용가능하다.
- 제약 조건이 까다로워 실제 사용하는 경우는 드물다고 한다.

### BranchPythonOperator

: 상황에 따라 뒤에서 실행되어야할 태스크를 동적으로 결정해주는 Operator

- 미리 정해둔 Operator중에 선택하는 형태
- Learn_BranchPythonOperator.py

```python
# 조건을 걸어줄 함수를 생성
def decide_branch(**context):
	current_hour = datetime.now().hour
	if current_hour < 12:
		return 'morning_task'
	else:
		return 'afternoon_task'

# BranchPythonOperator정의, python함수를 호출한다.
branching_operator = BranchPythonOperator(
	...
	python_callable=decide_branch,
	dag=dag
)
# branch의 결과에 따라서 실행되는 operator들
morning_task = EmptyOperator(
	task_id='morning_task'
)
afternoon_task= EmptyOperator(
	task_id='afternoon_task'
)

# 실행 순서 설정
branching_operator >> morning_task
branching_operator >> afternoon_task
```

- 실행되지 않는 task의 경우 skipped로 된다.

### LatestOnlyOperator

Time-sensitive한 task들이 과거의 backfill시 실행되는 것을 막기 위해 사용된다.
현재 시간이 execution_date보다 미래이고, 다음execution_date보다 과거인 경우에만 실행을 이어가고 아니면 중단된다. → 현재보다 과거의 경우에는 중단!

- 시간에 영향을 많이 받는 task들 앞에 사용하여 flow를 중단하는 역할을 수행한다.
- catchup=True 인 경우에 유용하게 사용 가능하다.

```python
from airflow.operators.latest_only import LatestOnlyOperator

with DAG(
	...
	t1 = EmptyOperator(task_id='task1')
	t2 = LatestOnlyOperator(task_id='latest_only')
	t3 = EmptyOperator(task_id='task3')
	t4 = EmptyOperator(task_id='task4')
)

t1 >> t2 >> [t3,t4]
```

### Trigger Rule

: Upstream task의 상황에 따라서 뒷단의 task의 실행 여부를 결정하기위해 사용

- trigger_rule 파라미터를 이용해서 결정 가능하다.
    - 가능한 값들 : ALL_SUCCESS (default), ALL_FAILED, ALL_DONE, ONE_FAILED, ONE_SUCCESS, NONE_FAILED, NONE_FAILED_MIN_ONE_SUCCESS
    - task의 상태는 success, fail, skip 3가지 상태가 존재하는 것에 유의해야한다.
    - airflow.utils.trigger_rule.TriggerRule을 가져와서 사용해야한다.

```python
# 예시 task1,2가 모두 성공해야 task3가 실행된다.
from airflow.utils.trigger_rule import TriggerRule

...
t1 = BashOperator(...)
t2 = BashOperator(...)
t3 = BashOperator(
	...
	trigger_rule=TriggerRule.ALL_DONE
)
[t1,t2] >> t3
```

---

## Task Grouping

: task들을 성격에 따라서 관리하는 경우에 용이하다.

- 다수의 파일 처리하는 DAG의 경우
    - 파일 다운로드, 파일 체크, 데이터 처리 ⇒ 3개의 태스크로 구성가능하다.
- TaskGroup 안에 TaskGroup nesting이 가능하다.
    - TaskGroup도 실행 순서 정의가 가능하다.

```python
from airflow.utils.task_group import TaskGroup

start = EmptyOperator(task_id="start")

with TaskGroup("Download", tooltip="Tasks for downloading daga") as section_1:
	task1 = ...
	task2 = ...
	...
	task_1 >> task2

	# nesting
	with TaskGroup(...) as inner_serction_2:
		...

start >> section_1 >> 
```

## Dynamic Dags

: 템플릿과 yaml을 기반으로 dag를 동적으로 만드려는 것. 비슷한 dag를 계속해서 매뉴얼하게 개발하는 것을 방지한다.

- 오너가 다르거나, 태스크의 수가 많아지는 경우에는 DAG를 복제하는 것이 좋다.
- 동작 구조: .yaml → template & generator → DAGs

### 예제

- yml

```yaml
# config_appl.yml

dag_id: 'APPL'
schedule: '@daily'
catchup: False
symbol: 'APPL'
```

- template.jinja2

```python
# templated_dag.jinja2

from airflow import DAG
from airflow.decorators import task
from datetime import datetime

with DAG(dag_id={% raw %}"get_price_{{ dag_id }}"{% endraw %},
	start_date = ...
	schedule = {% raw %}'{{ schedule }}" {% endraw %},
	catchup = {% raw %}{{ catchup or True }} {% endraw %} # catchup을 사용하거나 값이 없으면 True로 설정
) as dag:

@task
def extract(symlbol):
	return symbol
@task
def process(symbol):
	return symbol

@task
def store(symbol):
		return symbol

store(process(extract({% raw %} "{{ symbol }}" {% endraw %})))
```

- generator

```python
# generator.py
from jinja2 import Environment, FileSystemLoader
import yaml
import os

# 현재 실행중인 파일의 폴더의 절대 경로를 반환한다.
fire_dir = os.path.dirname(os.path.abspath(__file__))
env = Environment(loader=FileSystemLoader(file_dir))
template = env.get_template('templated_dag.jinja2')

for f in os.listdir(file_dir): # 파일 디렉토리 내에서 모든 파일 읽어오기
	if f.endswith(".yml"): # yml파일 읽기
		with open(f"{file_dir}/{f}","r") as cf: #yml 파일을 읽기모드로 열기
			config = yaml.safe_load(cf)
			with open(f"dags/get_price{config['dag_id']}.py","w") as f:  # 쓰기모드로 dag 생성
				# yml로 읽은 것을 template에 render 한 후 파일에 쓰는 작업 수행
				f.write(template.render(config))
```

### jekyll에서 jinja 출력하기...
: 다음 블로그 참고한다.. <br>
[{% raw %}jekyll에서 {{ }}, {% %}사용하기(escape liquid template)% endraw %}](https://atomic0x90.github.io/jekyll/markdown/2019/06/08/escape-liquid-template.html)


