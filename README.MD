# Airflow deployment on Openshift

![GitHub](https://img.shields.io/github/license/Veinar/airflow_openshift)
![Static Badge](https://img.shields.io/badge/contributions-welcome-blue)


This repository contains example Airflow deployment YAMLs that as whole allows user to deploy working system using only two commands (except creating namespace command) on Openshift. As database underlaying this component we are using Postgres (not scripted in this repo). With sidecar for pulling DAGs from git repository. We base on official Airflow docker image and use LDAP as authentication provider.

## Requirements

1. Access to Openshift cluster web console / API
1. You have to be an admin of namespace on which you are deploying Airflow.
1. Already deployed PostgreSQL with database for Airflow, with known username and password used to connect.

## Preparation / Configuration

In case you are willing to extend capabilities of base Airflow image by additional python libraries, please see `./build` directory where example BuildConfig is present. Remember to change image in deployments to match one written in BuildConfig in order Airflow to start.

**What needs to be done**:

Namespace across all YAMLs must be changed. Execute following command passing your namespace name in place of `your_namespace_name` inside this repository directory. 

```sh
find . -type f -name "*.yaml" -exec sed -i 's/\${namespace}/'"your_namespace_name"'/g' {} \;
```
To change the configuration file to suit your needs, please look at files listed below:
* `./airflow/airflow-config.yaml` - misc environmental variables 
* `./airflow/airflow-secrets.yaml` - misc environmental variables
* `./airflow/git-secrets.yaml` - git repository environmental variables
* `./airflow/webserver_config.py` - Airflow configuration file in which Auth provider is configured. Configure it to suit your needs using official Airflow documentation. **After configuration execute following command to create secret** from Python file:

```bash
oc create secret generic webserver_config --from-file=./airflow/webserver_config.py --dry-run=client -o yaml > ./airflow/webserver_config.yaml
```

## Deployment

To perform deploy, a tool named Openshift CLI aka `oc` or ability to create YAMLs from Cluster console is a must as mentioned in Requirements section above.

> Creating some objects from `serviceAccount` directory, especially `scc.yaml` will require cluster wide admin privileges.

* Using `oc` command (recommended):

```sh
oc project <your_namespace_name>
# Optional buildConfig applying
# oc apply -f ./build/* 
oc apply -f ./serviceAccount/*
oc apply -f ./airflow/*
```

* Using Openshift web console:

To deploy using YAML inside Openshift console just copy every YAML from this repository and divide them using `---` separator and paste it then execute.

## Tests

No testing is provided for this deployment.