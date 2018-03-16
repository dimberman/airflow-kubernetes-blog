# The Kubernetes "Whatever-your-heart-desires" Operator

## Introduction

Today, we are excited to announce a Kubernetes Operator to increase Apache Airflow's viability as a job orchestration engine using the power of the Kubernetes cloud deployment framework. 

Since its inception, Airflow's greatest strength has been its flexibility. Airflow offers a wide range of native operators for services ranging from Spark and HBase, to GCP and s3. Airflow also offers easy extensibility through its plugin framework. However, one limitation of the project is that Airflow users are confined to the frameworks and clients that exist on the Airflow worker at the moment of execution. If a user wishes to use a different version of scipy or test a new deep learning framework, they would need to either launch a new airflow cluster or risk conflicting with the dependencies of other users' workflows. 

To address this issue, we've utilized kubernetes to allow users to launch arbitrary docker containers and configurations. Airflow users can now have full power over their run-time environments, resources, and secrets, basically turning airflow into an "any job you want" scheduler.

## What is Kubernetes?

Before we go any further, let's take a moment for a quick overview of Kubernetes. [Kubernetes](https://kubernetes.io/) is an open-source container deployment engine released by Google. Based on Google's [Borg](http://blog.kubernetes.io/2015/04/borg-predecessor-to-kubernetes.html), kubernetes allows for easy deployment of images using a highly flexible API. Using kubernetes you can [deploy spark jobs](https://github.com/apache-spark-on-k8s/spark), launch end-to-end applications, or create multi-framework ETL pipelines using yaml, json, python, golang, or java bindings. The kubernetes API's programatic launching of containers seemed a perfect marriage with Airflow's "code as configuration" philosophy.

## The Kubernetes Operator

As DevOps pioneers, Airflow users are always looking for ways to make deployments and ETL pipelines simpler to manage. Any opportunity to decouple our pipeline steps while increasing monitoring can reduce future outages and fire-fights. The following is a list of benefits the Kubernetes Operator has in reducing the Airflow Engineer's footprint
* **Increased flexibility for deployments:**  
Airflow's plugin API has always offered a significant boon to engineers wishing to test new functionalities within their DAGS. On the downside, whenever a developer wanted to create a new operator, they had to develop an entirely new plugin. Now, any task that can be run within a Docker container is accessible through the exact same operator, with no extra Airflow code to maintain.
* **Flexibility of configurations and dependencies:** 
For operators that are run within static Airflow workers, dependency management can become quite difficult. If I want to run one task that requires [SciPy](https://www.scipy.org) and another that requires [NumPy](http://www.numpy.org), the developer would have to either maintain both dependencies within an Airflow worker or somehow configure ??
* **Usage of kubernetes secrets for added security:** 
Handling sensitive data is a core responsibility of any devops engineer. At every opportunity, airflow users want to minimize any API keys, database passwords, and login credentials to a strict need-to-know basis. With the kubernetes operator, users can utilize the kubernetes Vault technology to store all sensitive data. This means that the airflow workers will never have access to this information, and can simply request that pods be built with only the secrets they need

# Architecture

<img src="architecture.png">

The Kubernetes Operator uses the [Kubernetes Python Client](https://github.com/kubernetes-client/python) to generate a request that is processed by the APIServer (1). Kubernetes will then launch your pod with whatever specs you've defined (2). Images will be loaded with all the necessary environment variables, secrets and dependencies, enacting a single command. Once the job is launched, the operator only needs to monitor the health of track logs (3). Users will have the choice of gathering logs locally to the scheduler or to any distributed logging service currently in their Kubernetes cluster

# Examples

## Example 1: Running a basic container

In this first example, let's create a basic Docker image that runs simple Python\ commands. This example will only have two end-results: succeed or fail

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

This will create two pods on Kubernetes: one that has Python and one that doesn't. The Python pod will run the Python request correctly, while the one without Python will report a failure to the user.

<img src="image.png">

Link to github file

## Example 2: Running a model using SciPy

# Closing Statements

Final statements about all the possibilities this opens up

* Airflow Kubernetes
Executor
* Custom Deployments via python API
