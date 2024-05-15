# Camunda Terraform Red Hat OpenShift on AWS Modules

[![Camunda](https://img.shields.io/badge/Camunda-FC5D0D)](https://www.camunda.com/)
[![tests](https://github.com/camunda/camunda-tf-rosa/actions/workflows/tests.yml/badge.svg?branch=main)](https://github.com/camunda/camunda-tf-rosa/actions/workflows/tests.yml)
[![License](https://img.shields.io/github/license/camunda/camunda-tf-rosa)](LICENSE)

Terraform module which creates Red Hat OpenShift with Hosted Control Plane on AWS (ROSA HCP) cluster with an opinionated configuration targeting Camunda 8.

## Documentation

WIP

## Usage

### Requirements

* Terraform
* AWS CLI
* ROSA CLI
* OpenShift CLI


### Getting started : Create a ROSA HCP cluster

Base tutorial https://aws.amazon.com/blogs/containers/build-rosa-clusters-with-terraform/

#### I. Prepare the deployment

1. Login onto AWS
2. Check if ELB role exists
```bash
# To check if the role exists for your account, run this command in your terminal:
aws iam get-role --role-name "AWSServiceRoleForElasticLoadBalancing"

# If the role doesn't exist, create it by running the following command:
aws iam create-service-linked-role --aws-service-name "elasticloadbalancing.amazonaws.com"

```
3. Login onto [Red Hat Hybrid Cloud Console](https://console.redhat.com/openshift/token)
4. Generate an Offline token, click on "Load Token"
```bash
export RH_TOKEN=yourToken
rosa login --token=${RH_TOKEN}

rosa whoami

rosa verify quota --region="$AWS_REGION"

# this may fail due to org policy
rosa verify permissions --region="$AWS_REGION"

# TODO: check if this one is required:
rosa create account-roles --mode auto
```
5. Enable HCP ROSA on [AWS MarkePlace](https://docs.openshift.com/rosa/cloud_experts_tutorials/cloud-experts-rosa-hcp-activation-and-account-linking-tutorial.html)
    5.1 Navigate to the ROSA console : https://console.aws.amazon.com/rosa
    5.2 Choose Get started.
    5.3 On the Verify ROSA prerequisites page, select I agree to share my contact information with Red Hat.
    5.4 Choose Enable ROSA

Please note that **Only a single AWS account that will be used for service billing can be associated with a Red Hat account.**

#### II. Deploy a cluster with terraform

```bash
export ADMIN_PASS="yourPassword!!138"
export ADMIN_USER="kubeadmin"
export CLUSTER_NAME="rosatest"

terraform init
terraform plan -out rosa.plan -var "cluster_name=$CLUSTER_NAME" -var "htpasswd_password=$ADMIN_PASS" -var "htpasswd_username=$ADMIN_USER" -var "offline_access_token=$RH_TOKEN"
terraform apply rosa.plan
```

#### III. Retrieve cluster informations

1. In the output, you will have the created cluster id:
```bash
cluster_id = "2b3sq2r4geb7b6htaibb4uqk9qc9c3fa"
```
2. Describe the cluster
```bash
export CLUSTER_ID="2b3sq2r4geb7b6htaibb4uqk9qc9c3fa"

rosa describe cluster --output=json -c $CLUSTER_ID
```
3. Generate the kubeconfig:
```bash
export NAMESPACE="myNs"
export SERVER_API=$(rosa describe cluster --output=json -c "$CLUSTER_ID" | jq -r '.api.url')
oc login --username "$ADMIN_USER" --password "$ADMIN_PASS" --server=$SERVER_API

kubectl config rename-context $(oc config current-context) "$CLUSTER_NAME"
kubectl config use "$CLUSTER_NAME"

# create a new project
oc new-project "$NAMESPACE"
```

TODO: add modules doc

## Support

Please note that the modules have been tested with **[Terraform](https://github.com/hashicorp/terraform)** in the version described in the [.tool-versions](./.tool-versions) of this project.
