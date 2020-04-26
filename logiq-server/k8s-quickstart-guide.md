---
description: >-
  This page describes the LOGIQ deployment on Kubernetes cluster using HELM 3
  charts.
---

# K8S Quickstart guide

## 1 - Prerequisites

LOGIQ K8S components are made available as helm charts. Instructions below assume you are using HELM 3. Please read and agree [EULA](https://docs.logiq.ai/eula/eula) before proceeding.

### 1.1 Add LOGIQ helm repository

```bash
$ helm repo add logiq-repo https://logiqai.github.io/helm-charts
```

{% hint style="info" %}
The HELM repository will be named `logiq-repo`. For installing charts from this repository please make sure to use the repository name as the prefix e.g. 

`helm install <deployment_name> logiq-repo/<chart_name>`
{% endhint %}

You can now run `helm search repo logiq-repo` to see the available helm charts

```bash
$ helm search repo logiq-repo
NAME            	CHART VERSION	APP VERSION	DESCRIPTION
logiq-repo/logiq	1.0.0        	1.2.0      	A Helm chart for Kubernetes
```

### 1.2 Create namespace where LOGIQ will be deployed

```bash
$ kubectl create namespace logiq
```

This will create a namespace **`logiq`** where we will deploy the LOGIQ Log Insights stack. 

{% hint style="info" %}
If you choose a different name for the namespace, please remember to use the same namespace for the remainder of the steps
{% endhint %}

## 2. Install LOGIQ

```bash
$ helm install logiq --namespace logiq \
--set global.domain=logiq.my-domain.com logiq-repo/logiq
```

This will install LOGIQ and expose the LOGIQ UI and ingress ports at the domain `logiq.my-domain.com.` Replace `logiq.my-domain.com` with you own domain where the service will be hosted.

{% hint style="success" %}
You should now be able to login to LOGIQ UI at your domain using `https://logiq.my-domain.com` that you set in the ingress after you have updated your DNS server to point to the Ingress controller service IP

The default login and password to use is flash-admin@foo.com and flash-password. You can change these in the UI once logged in.
{% endhint %}

{% hint style="info" %}
The `logiq.my-domain.com` also fronts all the LOGIQ service ports as described in the [port details section](quickstart-guide.md#ports). 
{% endhint %}

![Logiq Insights Login UI ](../.gitbook/assets/screen-shot-2020-03-24-at-3.42.55-pm.png)

LOGIQ server provides Ingest, log tailing, data indexing, query and search capabilities.   
Besides the web based UI, LOGIQ also offers [logiqctl, LOGIQ CLI](https://docs.logiq.ai/logiq-cli) for accessing the above features.

## 3 Customizing the deployment

### 3.1 - Using an AWS S3 bucket

Depending on your requirements, you may want to host your storage in your own K8S cluster or create a bucket in a cloud provider like AWS.

#### 3.1.1 Create an access/secret key pair for creating and managing your bucket <a id="3-1-1"></a>

Go to AWS IAM console and create an access key and secret key that can be used to create your bucket and manage access to the bucket for writing and reading your log files

#### 3.1.2 Deploy the LOGIQ helm in gateway mode

Make sure to pass your `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` from [step 2.1.1](k8s-quickstart-guide.md#3-1-1) above and give a bucket name. The S3 gateway acts as a caching gateway and helps reduce API cost.

{% hint style="info" %}
You do not need to create the bucket, we will automatically provision it for you. Just provide the bucket name and access credentials in the the step below. 

If the bucket already exists, LOGIQ will use it. Check to make sure the access and secret key work with it.
{% endhint %}

```bash
$ helm install logiq --namespace logiq --set global.domain=logiq.my-domain.com \
--set global.environment.s3_bucket=<bucket_name>
--set s3-gateway.s3gateway.enabled=true \
--set s3-gateway.defaultBucket.enabled=true \
--set s3-gateway.defaultBucket.name=<bucket_name> \
--set s3-gateway.environment.AWS_ACCESS_KEY_ID=<access_key> \
--set s3-gateway.environment.AWS_SECRET_ACCESS_KEY=<secret_key> logiq-repo/logiq
```

{% hint style="info" %}
S3 providers may have restrictions on bucket name for e.g. AWS S3 bucket names are globally unique. 
{% endhint %}

### 3.2 - Install LOGIQ server certificates and Client CA `[OPTIONAL]`

LOGIQ supports TLS for all ingest. We also enable non-TLS ports by default. It is however recommended that  non-TLS ports not be used unless running in a secure VPC or cluster. The certificates can be provided to the cluster using K8S secrets. Replace the template sections below with your Base64 encoded secret files.

{% hint style="info" %}
If you skip this step, LOGIQ server automatically generates a ca and a pair of client and server certificates for you to use. you can get them from the ingest server pods under the folder `/flash/certs`
{% endhint %}

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: logiq-certs
type: Opaque
data:
  ca.crt: {{ .Files.Get "certs/ca.crt.b64" }}
  syslog.crt: {{ .Files.Get "certs/syslog.crt.b64" }}
  syslog.key: {{ .Files.Get "certs/syslog.key.b64" }}
```

Save the secret file e.g. `logiq-certs.yaml`. Proceed to install the secret in the same namespace where you want to deploy LOGIQ

```bash
$ kubectl apply --namespace logiq -f logiq-certs.yaml
```

The secret can now be passed into the LOGIQ deployment

```bash
$ helm install logiq --namespace logiq --set global.domain=logiq.my-domain.com \
--set logiq-flash.secrets_name=logiq-certs logiq-repo/logiq
```

## 4 Tear down

If and when you want to decommission the installation use following commands

```bash
$ helm delete logiq --namespace logiq
$ helm repo remove logiq-repo
$ kubectl delete namespace logiq
```

If you followed installation steps in section 3.1 - Using an AWS S3 bucket, you may want to delete the s3 bucket that was specified at deployment time.

