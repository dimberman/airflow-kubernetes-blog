
# Introduction



## What is kubernetes?

Paragraph on the background of kubernetes

Paragraph on 

## The kubernetes operator: Launch whatever you want

## Example 1: Running a basic container

For this first example, let's create a basic docker container that runs simple python commands. This container should be able to pass and fail at will, and record logs that show up in the airflow terminal.


```python
import sys


def fail():
    return 10 / 0


def succeed():
    return 10 / 1


if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: wordcount <file>", file=sys.stderr)
        exit(-1)
    arg = sys.argv[1]
    if arg == 'pass':
        succeed()
    elif arg == 'fail':
        fail()
    else:
        print('invalid command: {}'.format(arg))
        exit(-1)
```

Paragraph discussing creating a docker container for running this scipy code using an entry point.


```python
FROM ubuntu:16.04

# install python dependencies
RUN apt-get update -y && apt-get install -y \
        wget \
        python-dev \
        python-pip \
        libczmq-dev \
        libcurlpp-dev \
        curl \
        libssl-dev \
        git \
        inetutils-telnet \
        bind9utils

RUN pip install -U setuptools && \
    pip install -U pip


COPY airflow-example.py /airflow-example.py
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
```

Discussion of entrypoint files 


```python
#!/usr/bin/env bash

python /airflow-example.py
```


```python
# Airflow DAG File that creates two kubernetes operators, one that passes and one that fails

passing = KubernetesPodOperator(namespace='default',
                          image="airflow-test",
                          arguments=["succeed"],
                          labels={"foo": "bar"},
                          name="passing-test",
                          task_id="task1"
                          )

failing = KubernetesPodOperator(namespace='default',
                          image="airflow-test",
                          arguments=["succeed"],
                          labels={"foo": "bar"},
                          name="fail",
                          task_id="task1"
                          )

failing.set_upstream(passing)
```

### This will eventually be a series of images about the running DAGs

<img src="files/image.png">

Link to github file

## Example 2: Running a model using scipy

## How it Works

# Closing Statements

Final statements about all the possibilities this opens up
