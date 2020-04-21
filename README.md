# hyper-kube-config

[![Build Status](https://travis-ci.org/zillow/hyper-kube-config.svg?branch=master)](https://travis-ci.org/zillow/hyper-kube-config)

[![PyPI version](https://badge.fury.io/py/hyper-kube-config.svg)](https://badge.fury.io/py/hyper-kube-config)

hyper-kube-config - Provides a secure [Serverless](https://serverless.com/) API to store and retrieve [Kubernetes cluster config credentials](https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/). hyper-kube-config leverages [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/index.html) for storing credential information. Included is a [kubectl plugin](https://kubernetes.io/docs/tasks/extend-kubectl/kubectl-plugins/) to interface with hyperkube API. 

You can also set cluster's status and retrieve this information. Useful in CI/CD pipelines when trying to pull access to particular cluters without needing to know their names, just their environments. 

It requires a configuration file. See [hyperkube-config.yaml.example](hyperkube-config.yaml.example) for layout.

![hyper-kube-config](https://user-images.githubusercontent.com/538171/46702337-8edc2780-cbd7-11e8-8ba5-dbbe9916708a.png)


## Install hyperkube kubectl plugin

```bash
pip3 install hyper-kube-config 
```

## Setup `~/hyperkube-config.yaml` file

The default locations for the config file is `~/hyperkube-config.yaml`. You can also place the config file at a different location and pass the location as a command line option `-c <hyper-kube-config-location>` or `--config <hyper-kube-config-location>`


## Post cluster and creds to hyperkube store

```bash
kubectl hyperkube add --k8s-config ~/.kube/config
```

## Remove cluster and creds

```bash
kubectl hyperkube remove --cluster-to-remove 'k8s-cluster-example.cloud' 
```

## Get user creds and merge it with existing `~/.kube/config`

```bash
# for single cluster
kubectl hyperkube get --cluster cloud-infra.cloud -m
```

## Get user creds multiple clusters and merge them with existing `~/.kube/config`

```bash
kubectl hyperkube get \
  --cluster cloud-infra.cloud \
  --cluster bar-cluster.cloud \
  --cluster baz-cluster.com -m
```

## Get creds for all clusters and merge it with existing `~/.kube/config`

```bash
kubectl hyperkube get-all -m
```

## List clusters

```bash
kubectl hyperkube list
```

## Store and Associate SSH PEM and CA private key with Cluster

#### Store SSH Pem

```bash
kubectl hyperkube add-pem --pem ~/.ssh/my-cluster.pem 
```

#### Get Stored SSH Pem

```bash
kubectl hyperkube get-pem --cluster my-cluster.net
```


#### Store Add CA Private Key

```bash
kubectl hyperkube add-ca-key --ca-key ca-key-file.key --cluser my-cluster.net
```

## Set Cluster Status and/or Environment References

```bash
# Set arbitrary status string and environment reference for given cluster
kubectl hyperkube set-cluster-status --cluster my-cluster.net --status active --environment stage
```

## Get Cluster Status for Environment

```bash
# Returns list of clusters that are active for prod environment
kubectl hyperkube get-cluster-status --status active --environment prod
```

## Requirements

* [Serverless](https://serverless.com/) - Serverless Framework
* [Docker](https://docker.com) - For serverless deploy
* [serverless-python-requirements](https://www.npmjs.com/package/serverless-python-requirements) plugin. Uses Docker and Pip to package a newer vesion of Boto3 for AWS Lambda function use. AWS Lambda boto3 version by default doesn't have AWS Secrets Manager support for tags.
* [click](https://click.palletsprojects.com/en/7.x/) - for hyperkube kubectl plugin
* [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) - version 1.12 or higher recommended for stable plugin support.

### Deploying Serverless API

```bash
sls deploy \
  --stage dev \
  --product k8s \
  --owner myteam@foo.cloud \
  --team myteam \
  --environment dev
```
This will launch your hyperkube API. Capture the API URL, api key and stage for your hyperkube.yaml configuration. The `kubectl hyperkube` commands will leverage the config to interact with your stored k8s configs.

Serverless will launch an [AWS API Gateway](https://docs.aws.amazon.com/apigateway/index.html) to handle API requests forwardered to [AWS Lambda functions](https://docs.aws.amazon.com/lambda/index.html#lang/en_us). A Dynamodb table is configured to store non-senstative cluster config details, while sensative information in uploaded configs (passwords and certs) is stored in [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/index.html).

### Testing for lint errors on your local machine

1. Install Test Dependencies

  ```
  pip install -U -r tests/requirements.txt
  ```

2. Run flake8 to check for lint errors

  ```
  flake8 *.py tests cli/kubectl-hyperkube
  ```

3. Run unit tests

  ```
  python -m unittest discover -s tests/ -p "*.py"
  ```
