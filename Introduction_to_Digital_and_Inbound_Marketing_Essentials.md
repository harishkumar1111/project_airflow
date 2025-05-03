# Introduction to Apache Airflow: Automating and Managing Workflows

## Introduction  
Apache Airflow is an open-source platform designed to programmatically author, schedule, and monitor workflows. In today’s data-driven world, automating complex processes—such as data pipelines, machine learning model training, and ETL (Extract, Transform, Load) tasks—is critical for efficiency and scalability. Airflow allows teams to define workflows as code, making them reusable, dynamic, and easily maintainable. Whether orchestrating daily data ingestion or coordinating multi-team workflows, Airflow provides the tools to ensure tasks run in the right order, at the right time, and with proper monitoring.

---

## Learning Objectives  
1. **Understand** the core components of Apache Airflow, including DAGs, Operators, and Tasks.  
2. **Design** a basic workflow using Airflow’s Python-based framework.  
3. **Monitor** and troubleshoot workflows using Airflow’s UI and logging features.  

---

## What is a Workflow?  
A workflow is a sequence of tasks that are executed in a specific order to achieve a goal. In data engineering, workflows might include steps like fetching data, cleaning it, analyzing it, and generating reports. Manually managing these processes is error-prone and inefficient. Apache Airflow solves this by allowing you to define workflows as code, automating their execution and monitoring.  

---

## Core Components of Apache Airflow  

### 1. Directed Acyclic Graph (DAG)  
- **Core Idea**: A DAG defines the workflow structure, representing tasks and their dependencies. It ensures tasks run in the correct order without cycles.  
- **Key Features**:  
  - Written in Python, making it dynamic and flexible.  
  - Scheduled using cron-like syntax or time intervals.  
  - Enables task reuse and modularity.  

# Directed Acyclic Graph (DAG) in Apache Airflow

A **Directed Acyclic Graph (DAG)** is the backbone of Apache Airflow's workflow management system. It represents a collection of all the tasks you want to run, organized in a way that reflects their relationships and dependencies.

## Core Concept

- **Directed**: The workflow has a specific flow from one task to another.
- **Acyclic**: No cycles are allowed; a task cannot be dependent on itself either directly or indirectly.
- **Graph**: Composed of nodes (tasks) and edges (dependencies), forming a structure that defines execution order.

This structure ensures that tasks execute in a valid, logical sequence, preventing circular dependencies and allowing for deterministic workflows.

## Key Features

- **Python-Based**: DAGs are defined in standard Python files, allowing the use of dynamic and programmatic structures.
- **Flexible Scheduling**: DAGs can be scheduled using cron expressions or defined time intervals, making them suitable for a wide range of use cases.
- **Task Modularity and Reusability**: Tasks and operators can be reused across different DAGs, supporting DRY (Don't Repeat Yourself) principles.
- **Clear Dependency Management**: Dependencies between tasks are explicitly defined, ensuring predictable execution flows.
- **Retry and Alert Mechanisms**: DAGs support configurable retry strategies and failure notifications to enhance reliability.
- **Backfilling and Catch-up**: Missed runs can be automatically executed when a DAG is re-enabled or updated, ensuring data consistency over time.

## DAG Characteristics

- **Declarative Structure**: You define what should be done and in what order, rather than how to do it step by step.
- **Dynamic Creation**: DAGs can be created dynamically using loops or external configuration files, enabling scalable and templated workflows.
- **Configurability**: Settings such as owner, retry logic, start date, and timeout behavior can be easily customized per DAG or task.
- **Visibility and Monitoring**: The Airflow UI provides a visual representation of DAGs, task statuses, execution times, and logs, aiding in debugging and management.

## Best Practices

- **Name DAGs and Tasks Clearly**: Use descriptive names to make workflows self-explanatory.
- **Keep DAGs Lightweight**: Avoid heavy computations or API calls in the DAG definition file itself.
- **Avoid Cyclic Dependencies**: Always ensure that the task dependency graph remains acyclic.
- **Separate Logic from Configuration**: Encapsulate business logic in external scripts or modules to keep DAGs clean and readable.
- **Limit DAG File Complexity**: Keep each DAG focused and manageable; if it gets too large, consider breaking it into multiple DAGs.

---

By leveraging DAGs effectively, Airflow users can design scalable, maintainable, and robust data workflows tailored to their operational needs.



## Operators

**Core Idea**: Operators determine what a task does. Airflow provides pre-built operators for common actions (e.g., `PythonOperator`, `BashOperator`, `EmailOperator`).

**Example**: Use the `BashOperator` to run a shell script:

```python
from airflow.operators.bash import BashOperator

process_data = BashOperator(
    task_id="process_data",
    bash_command="python /scripts/transform.py"
)
```

## Tasks

**Core Idea**: Tasks are individual units of work within a DAG. Each task is an instance of an operator.

**Key Features**:
- Tasks can depend on other tasks using `>>` or `set_downstream()`
- Retries and error handling are configurable per task

**Example Task Dependencies**:
```python
extract >> process_data  # Ensures 'process_data' runs after 'extract'
```

## Airflow Architecture: How It Works

Airflow's modular architecture includes:
- **Scheduler**: Triggers workflows and submits tasks to the executor
- **Executor**: Runs tasks (locally or on distributed systems like Kubernetes)
- **Web Server**: UI for monitoring DAGs, inspecting logs, and managing tasks
- **Metadata Database**: Stores DAG definitions, task states, and execution history

![Airflow Architecture Diagram](https://airflow.apache.org/docs/apache-airflow/stable/_images/arch-diag-basic.png)

## Use Cases for Apache Airflow

1. **Data Pipelines**: Automate ETL processes across databases and cloud services (e.g., AWS S3, Snowflake)
2. **Machine Learning**: Schedule model training, data validation, and deployment tasks
3. **DevOps Automation**: Run infrastructure checks, backups, and deployment scripts
4. **Business Reports**: Generate daily/weekly analytics reports

## Building a Workflow: 5-Step Guide

| Step | Description | Example |
|------|-------------|---------|
| 1. Define Goals | Identify tasks, dependencies, and scheduling needs | "Load daily sales data by 5 AM" |
| 2. Write DAG File | Structure tasks in Python | Store in `dags/` folder |
| 3. Test DAG | Validate tasks | `airflow tasks test my_dag extract_task 2024-01-01` |
| 4. Deploy & Monitor | Use Airflow UI | Check Graph View for progress |
| 5. Handle Failures | Configure error handling | Retry 3x + email alerts |

## Key Metrics to Monitor
- ✅ DAG Execution Time
- ✅ Task Success Rate
- ⚠️ Scheduler Latency
- 📈 Resource Utilization (CPU/Memory)

## Lesson Summary
1. **Core Components**: DAGs + Operators + Tasks
2. **Flexibility**: Python-based workflow design
3. **Scalability**: Kubernetes support
4. **Monitoring**: Real-time UI insights

## Challenge: Create API Data Pipeline DAG

**Requirements**:
```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator
from airflow.operators.email import EmailOperator
from datetime import datetime

def fetch_data():
    print("Fetching data from API...")

def process_data():
    print("Processing data...")

with DAG(
    "api_data_pipeline",
    start_date=datetime(2024, 1, 1),
    schedule_interval="@daily",
    default_args={"retries": 2}
) as dag:
    fetch = PythonOperator(task_id="fetch_data", python_callable=fetch_data)
    process = PythonOperator(task_id="process_data", python_callable=process_data)
    upload = BashOperator(task_id="upload_to_s3", bash_command="aws s3 cp /data/output.csv s3://my-bucket/")
    notify = EmailOperator(
        task_id="send_email",
        to="team@example.com",
        subject="Pipeline Status",
        html_content="Success!"
    )

    fetch >> process >> upload >> notify
```

🔗 **Resources**:  
[Official Documentation](https://airflow.apache.org/docs/) | 
[YouTube Tutorial](https://www.youtube.com/watch?v=K9AnJ9_ZAXE)
```

