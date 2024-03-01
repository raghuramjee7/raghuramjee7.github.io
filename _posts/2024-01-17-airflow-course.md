---
layout: post
title: "An Airflow Crash Course"
categories: junk
author:
- Raghuramjee Janapareddy
meta: "Springfield"
---

# An Airflow Crash Course

## Setup

1. Create a git directory for practise and clone it
2. Create a pipenv environment and install req dependencies - `pipenv install --python=3.7 Flask==1.0.3 apache-airflow==1.10.3`
3. We setup airflow home dir in an env file - `echo "AIRFLOW_HOME=${PWD}/airflow" >> .env`
4. Airflow req a db to run, by default it uses a sqlite db for this process - `airflow initdb`
5. Then we create a folder to setup our dags - `mkdir -p ${AIRFLOW_HOME}/dags/`
6. Run airflow - `airflow webserver -p 8081`
7. Start airflow scheduler - `airflow scheduler`
8. Run task from a dag - `airflow run <dag> <task> 2020-05-31`
9. List tasks in a dag - `airflow list_tasks <dag>`
10. Pause and unpause dag - `airflow pause/unpause <dag>`

### Workflow
1. Workflow is a sequence of events, in airflow we represent workflow as a directed acyclic graph
2. A task is a unit of work in a dag. It is represented as a node in graph
3. The goal of a task is to achieve something, the method it uses to achieve it is called an operator. eg BashOperator, PythonOperator, Custom Operators etc
4. A task instance is the execution of a task run at a date


## DAGs
In airflow, DAGs are defined as python files in the `dags/` folder.

### First DAG with BashOperator

```
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime, timedelta

# Common Parameters to intiate the operator
default_args = {
    "owner": "raghu",
    "retries": 5,
    "retry_delay": timedelta(minutes=5),
}

# Create an instance of DAG
with DAG(
    dag_id = "my_first_dag_v1", # id of dag
    description = "demo first dag", # description
    default_args = default_args, # default parameters of dag
    # parameters for start date and interval
    start_date = datetime(2024, 2, 1, 2), # start from 2024 feb 1 everyday at 2am
    schedule_interval = "@daily", # everyday

) as dag:
    task1 = BashOperator(
        task_id = "first_task",
        bash_command = "echo This is the first task"
    )
    task2 = BashOperator(
        task_id = "second_task",
        bash_command = "echo This is the second task, this is run after the first task"
    )
    task3 = BashOperator(
        task_id = "thid_task",
        bash_command = "echo This is the third task, this is run after the first task, parallel to second task"
    )

    # This is one way to setup the dependencies
    task1.set_downstream(task2)
    task1.set_downstream(task3)

    # This is another way to setup the dependencies
    task1 >> task2
    task1 >> task3

    # This is the last way
    task1 >> [task2, task3]
```

### DAG with PythonOperator
```
from airflow.operators.python import PythonOperator

def greet(name, age):
    print(f"My name is {name} and I am {age} years old")

task_1 = PythonOperator(
        task_id = "greet_fn",
        python_callable = greet,
        op_kwargs = {"name": "Raghu", "age": 22}
    )

    task_1
```


### XComs
1. We can share data between tasks using XComs
2. Basically we push data from one task to XCom and pull it from there by another task
3. By default, all return values from functions are pushed to XComs
4. We do not use XComs to share large data, we use it to share metadata, the max limit of xcom is 48kb, although it varies with the backend db
5. We use `ti` or also called, task instance object to pusha and pull data from XComs, we use xcom_push and xcom_pull methods of ti object.
```
from airflow.operators.python import PythonOperator

def greet(ti):
    # pull data from ti
    name = ti.xcom_pull(task_ids = "get_name", key = "name")
    age = ti.xcom_pull(task_ids = "get_age", key = "age")

    print(f"My name is {name} and I am {age} years old")

def returns_name(ti):
    # we can simply return like this return {"name": "Raghu"}
    # push data
    ti.xcom_push(key="name", value="Raghu")

def returns_age(ti):
    # push data
    ti.xcom_push(key="age", value=22)

task_1 = PythonOperator(
        task_id = "get_name",
        python_callable = returns_name,
    )
    task_2 = PythonOperator(
        task_id = "get_age",
        python_callable = returns_age,
    )
    task_3 = PythonOperator(
        task_id = "greet",
        python_callable = greet,
    )

    [task_1, task_2] >> task_3
```

### TaskFlow API
1. We use taskflow api to reduce the number of lines of code that we use.
2. Sample DAG with TaskFlow API - 
```
from airflow.decorators import dag, task
from datetime import datetime, timedelta

default_args = {
    "owner": "raghu",
    "retries": 5,
    "retry_delay": timedelta(minutes=5),
}

# we use the dag decorator to define the DAG
@dag(
    dag_id = "taskflow_api_dag_v1",
    description = "My DAG with taskflow api",
    default_args= default_args,
    start_date = datetime(2024, 2, 1, 2),
    schedule_interval = "@daily",
)
def hello_world_etl():
    
    # we use the task decorator to define the task
    # we define that multiple outputs are returned here
    @task(multiple_outputs = True)
    def get_name():
        return {
            "first_name": "raghu",
            "last_name": "ramjee"
        }

    @task()
    def get_age():
        return 22
    
    @task()
    def greet(first_name, last_name, age):
        print(f"My name is {first_name} {last_name} and I am {age} years old")
        
    name = get_name()
    age = get_age()
    greet(first_name=name["first_name"], 
          last_name=name["last_name"], 
          age=age)

greet_dag = hello_world_etl()
```

### Catchup and Backfill
1. In Airflow we have two concepts, catchup and backfill
2. Catchup or Backfill is the process of running all the tasks that have been missed while the dag was paused
3. When we define the DAG, we have a parameter called catchup, which is set to True by default. We can set it to False to disable catchup - `catchup = False`
4. By default, catchup is set to True, which means that all the tasks that have been missed will be run when the dag is unpaused
5. We can also use the backfill command to run the missed tasks - `airflow dags backfill -s <start_date> (eg - 2023-12-01) -e <end_date> <dag_id>`

### CRON Expressions
1. We can use cron expressions, timedetla objects to define the schedule interval
2. Use crontab guru to check cron expressions meaning

### Connect with Postgres
1. Go to Admin -> Connections to create a new connection
2. We can setup all the values of the connection, if the postgres connection type is not avaialable, we need to install the postgres provider - `pipenv install apache-airflow-providers-postgres` and restart the webserver.
3. Once the connection is created, we can use that connection_id to interact with the db.
```
from airflow import DAG
from airflow.providers.postgres.operators.postgres import PostgresOperator
from datetime import datetime, timedelta

# Common Parameters to intiate the operator
default_args = {
    "owner": "raghu",
    "retries": 5,
    "retry_delay": timedelta(minutes=5),
}

# Create an instance of DAG
with DAG(
    dag_id = "dag_with_postgres_operator", # id of dag
    description = "demo first dag", # description
    default_args = default_args, # default parameters of dag
    # parameters for start date and interval
    start_date = datetime(2024, 2, 1, 2), # start from 2024 feb 1 everyday at 2am
    schedule_interval = "@daily", # everyday
    catchup = False,

) as dag:
    task = PostgresOperator(
        task_id = "create_table_in_pg_db", # id of task
        postgres_conn_id = "postgres_db_connection", # connection id of postgres
        sql = "create table if not exists dag_runs (name varchar(50), age int);", # sql query
    )
    task2 = PostgresOperator(
        task_id = "insert_data_into_pg_db", # id of task
        postgres_conn_id = "postgres_db_connection", # connection id of postgres
        sql = "insert into dag_runs values('Raghu', 25);", # sql query
    )
    task >> task2
```
