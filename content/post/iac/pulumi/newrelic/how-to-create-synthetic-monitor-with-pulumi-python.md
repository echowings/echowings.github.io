---
title: "How to Create Synthetic Monitor With Pulumi Python"
date: 2024-03-17T11:39:49+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=d16575c4
categories:
  - IAC
  - DEVOPS
  - SRE
  - CLOUD ENGINEERING
  - PYTHON
  - Pulumi
  - NEW RELIC
tags:
  - iac
  - sre
  - devops
  - cloud engineering
  - new relic
  - monitor
  - pulumi
  - python
slug:
  - pulumi
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---

## Prepared environment
### Install pulumi
### install python
### install poetry

## Create pulumi project
### Create pulumi project

```shell
## create folder
mkdir newrelic-pulumi-example && cd newrelic-plumi-example
# crate pulumi project without active python venv
pulumi new python --name pulumi-newrelic-simple-mintor --stack dev --description "implement new relic sythetic simple monitoring" -y --generate-only --force

## Chang python virtual environment path 
sed -i 's/venv/.venv/g' Pulumi.yaml

# init poetry
poetry init

# install pulumi-newrelic module
poetry add pulumi-newrelic

# Install poetry
poetry install --no-root

# active virtual environment
source "$( poetry env list --full-path | grep Activated | cut -d' ' -f1 )/bin/activate"
```

### Modify __main__.py

```shell
tee  __main__.py << "EOF"
"""A Python Pulumi program"""

import pulumi
import pulumi_newrelic as newrelic

# Create a New Relic Synthetic Monitor
synthetic_monitor = newrelic.synthetics.Monitor("example-monitor",
        uri = "https://echowings.github.io",
        type = "SIMPLE",
        period = "EVERY_MINUTE",
        status = "ENABLED",
        locations_publics = ["AWS_US_WEST_1", "AWS_EU_WEST_2" ] # Locations from where thechecks are mode
        )
# Export the ID of the monitor
pulumi.export('monitor_id', synthetic_monitor.id)
EOF
```

### Set pulumi api

```shell
pulumi config set newrelic:apiKey xxxx
pulumi config set newrelic:accountId xxxx
```

###  Create dev stack 
```shell
pulumi stack init dev --yes
```

### Preview and check pulumi code
```shell
pulumi preview 
```

### Pulumi deploy
```shell
pulumi up --yes
```
### Pulumi destroy

```shell
pulumi destroy --yes
```

### Pulumi delete stack

```shell
pulumi stack rm dev --yes
```

## Reference
  - [how to active poetry venv](https://stackoverflow.com/questions/60580332/poetry-virtual-environment-already-activated)
  - [pulumi synthetics example](https://www.pulumi.com/registry/packages/newrelic/api-docs/synthetics/monitor/)
  - [create a new relic monitor using python](https://www.pulumi.com/ai/answers/5MKZJ3agrJDTQQuXEsNFD6/creating-a-new-relic-monitor-using-python)
