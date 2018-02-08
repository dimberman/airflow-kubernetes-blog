# Introduction

Today, we are excited to announce a Kubernetes Operator to increase airflow's
viability as a job orchestration engine using the power of the Kubernetes cloud
deployment framework. 

Since its inception, airflow's power as a job
orchestrator has been its flexibility. Airflow offers a wide range of native
operators (for services such as spark, hbase, etc.) while also offering easy
extensibility through its plugin framework. However, one limitation of the
Apache Airflow project is that...

Over the next few months, we will be offering
a series of kubernetes-based offerings that will vastly expand airflows' native
capabilities. With the addition of a Kubernetes Operator, users will be able to
launch arbitrary docker containers with customizable resources, secrets, and...

## What is kubernetes?

[Kubernetes](https://kubernetes.io/) is an open-source container deployment
engine released by Google. Based on google's own
[Borg](http://blog.kubernetes.io/2015/04/borg-predecessor-to-kubernetes.html),
kubernetes allows for easy deployment of images using a highly flexible API.
Using kubernetes you can [deploy spark jobs](https://github.com/apache-spark-
on-k8s/spark), launch end-to-end applications, or ... using yaml files, python,
golang, or java bindings. The kubernetes API's programatic launching of
containers seemed a perfect marriage with Airflow's "code as configuration"
philosophy.

# The Kubernetes Operator: The "Whatever-your-heart-desires" Operator

As DevOps
pioneers, we are always looking for ways to make our deployments and ETL
pipelines simpler to manage. Any opportunity to reduce the number of moving
parts in our codebase will always lead to future opportunities to break in the
future. The following are a list of benefits the Kubernetes Operator in reducing
the Airflow Engineer's footprint

* **Increased flexibility for deployments**
Airflow's plugin API has always offered a significant boon to engineers wishing
to test new functionalities within their DAGS, however it has always had the
downside that to create a new operator, one must develop an entirely new plugin.
Now any task that can be run within a docker container is accessible through the
same same operator with no extra airflow code to maintain.
* **Flexibility of
configurations and dependencies** For operators that are run within static
airflow workers, dependency management can become quite difficult. If I want to
run one task that requires scipy, and another that requires numpy, I have to
either maintain both dependencies within my airflow worker, or somehow configure
* **Usage of kubernetes secrets for added security** Handling sensitive data is
a core responsibility of any devops engineer. At every opportunity, we want to
minimize any API keys, database passwords,  or ... to a strict need-to-know
basis. With the kubernetes operator, we can use the kubernetes Vault technology
to store all sensitive data. This means that the airflow workers will never have
access to this information, and can simply request that pods be built with only
the secrets they need

# Examples

## Example 1: Running a basic container

For this first example, let's create a basic docker image that runs simple
python commands. This example will only have two end-results: Succeed or fail.

To run this code, we'll create a DockerFile and an `entrypoint.sh` file that
will run our basic python script (while this entrypoint script is pretty simple
at the moment, we will later see how it can expand to more complex use-cases).
With the following [Dockerfile](https://github.com/dimberman/airflow-kubernetes-
operator-example/blob/master/Dockerfile) and
[entrypoint](https://github.com/dimberman/airflow-kubernetes-operator-
example/blob/master/entrypoint.sh) We have a working image to test!

### Integrating into Airflow

To test our new image, we create a simple airflow
DAG that will run two steps: passing and failing

```{.python .input}
from airflow import DAG
from datetime import datetime, timedelta
from airflow.contrib.operators.kubernetes_pod_operator import KubernetesPodOperator
from airflow.operators.dummy_operator import DummyOperator


default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime.utcnow(),
    'email': ['airflow@example.com'],
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5)
}

dag = DAG(
    'kubernetes_sample', default_args=default_args, schedule_interval=timedelta(minutes=10))


start = DummyOperator(task_id='run_this_first', dag=dag)

passing = KubernetesPodOperator(namespace='default',
                          image="airflow/ci:latest",
                          cmds=["python","-c"],
                          arguments=["print('hello world')"],
                          labels={"foo": "bar"},
                          name="passing-test",
                          task_id="passing-task",
                          get_logs=True,
                          dag=dag
                          )

failing = KubernetesPodOperator(namespace='default',
                          image="ubuntu:1604",
                          cmds=["python","-c"],
                          arguments=["print('hello world')"],
                          labels={"foo": "bar"},
                          name="fail",
                          task_id="failing-task",
                          get_logs=True,
                          dag=dag
                          )

passing.set_upstream(start)
failing.set_upstream(start)
```

This will create two pods on kubernetes: one that has python and one that
doesn't. The python pod will run the python request correctly, while the one
without python will report a failure to the user.

<img src="files/image.png">

Link to github file

## Example 2: Running a model using scipy

# Architecture

<img src="architecture.png">

1. The kubernetes operator uses the [Kubernetes
Python Client](https://github.com/kubernetes-client/python) to generate a
request that is processed by the APIServer
2. Kubernetes will then launch your
pod with whatever specs you wish. Images will be loaded with all necessary
environment varialbes, secrets, dependencies, and enact a single command.
3.
Once the job is launched, the only things that the operator needs to do are
monitor the health of track logs. Users will have the choice of gathering logs
locally to the scheduler or any distributed logging service currently in their
kubernetes cluster

# Closing Statements

Final statements about all the possibilities this opens up

* Airflow Kubernetes
Executor
* Custom Deployments via python API
