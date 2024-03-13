---
title: "How to Depoy Flask App on Docker With Pulumi"
date: 2024-03-13T16:08:20+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=99bb0d91
categories:
  - iac
  - pulumi
  - gitops
tags:
  - pulumi
  - devops
  - iac
  - python
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---


## Create flask app

### Create project directory

```shell
mkdir  -p pulumi-docker/flask-first-steps/app
```
### create Dockerfile
```shell
tee pulumi-docker/flask-first-steps/Dockerfile << "EOF"
FROM tiangolo/uwsgi-nginx-flask:python3.8
COPY ./app /app
EOF
```

### Create main.py
```shell
tee pulumi-docker/flask-first-steps/app/main.py << "EOF"
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'

if __name__ == "__main__":
    # Only for debugging while developing
    app.run(host='0.0.0.0', debug=True, port=80)
EOF
```

### Option: Run app in docker
```bash
cd 
docker build -t myflaskservice .
docker run -d --name myflaskservice -p 8080:80 myflaskservice
curl 127.0.0.1:8080
```

## Create Pulumi project
### Create pulumi project at the same folder level

```bash
mkdir  -p pulumi-docker/pulumi-first-steps/
cd pulumi-docker/pulumi-first-steps/
pulumi login --local
pulumi new python
```

### change requirements.txt

```shell
tee requirements.txt << "EOF" 
pulumi>=2.0.0,<3.0.0
pulumi-docker
protobuf==3.20.*
EOF
```
### Active venv and install python modules
```shell
source venv/bin/activate
pip3 install -r requirements.txt
```

###  Modify `__main__.py`

```bash
tee __main__.py << "EOF"
"""A Python Pulumi program"""

import pulumi
import pulumi_docker as docker


myapp_build = docker.DockerBuild(context=f'../flask-first-steps')
myapp_image = docker.Image("myapp_image", image_name='myapp', build=myapp_build, skip_push=True)

container_ports = docker.ContainerPortArgs(internal=80, external=8080)
myapp_container = docker.Container("myapp_container", image="myapp", ports=[ container_ports])
EOF
```

## container get started
```bash
export PULUMI_CONFIG_PASSPHRASE=""
pulumi up --yes
```

## Check docker

```bash
docker ps
curl localhost:8080
```
### Clean up
```bash
pulumi destroy --yes
pulumi stack rm dev --yes
```

## Reference
  - [First Steps with Flask. A Quick Tutorial to Deploy a Flask… | by Tobias Wissmueller | Medium](https://twissmueller.medium.com/first-steps-with-flask-dcf7325ad2c6)
  - [First Steps with Pulumi. The most minimal and functional… | by Tobias Wissmueller | Level Up Coding](https://levelup.gitconnected.com/first-steps-with-pulumi-e6025eecece4)
