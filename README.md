# Application CD Workflow

<p>&nbsp;</p>

## Overview

<p>&nbsp;</p>

This is made to deploy to the environment built by the repo `appdev-infra` via Terraform.

<p>&nbsp;</p>

This workflow will create and deploy an application to Cloud Run using best practise security methods.


* Vulnerability Scanning of the application

* Binary Authorization tagging

For this deployment, I am using Google's Cloud Run "Hello" container

<p>&nbsp;</p>


## Prerequisite

From the IaC outputs, the following details need to be updated in the workflow for the application deployment:

`cd.yml` 

* BUCKET_PREFIX _(dev)_

* SERVICE_ACCOUNT  _(sa-gha-appdev-cm-dev@appdev-cm-dev.iam.gserviceaccount.com)_

* WORKLOAD_IDENTITY_PROVIDER _(projects/37532543929/locations/global/workloadIdentityPools/github-action-pool-dev/providers/github-actions-provider)_

* DEV_PROJECT_NUMBER _(37532543929)_

* DEV_PROJECT_ID _(appdev-cm-dev)_

<p>&nbsp;</p>

The image name and resource name need to be decalred as environment variables.

* IMAGE_NAME

* RESOURCE_CLOUD_RUN (This must match the name previously defined in the deployment by `appdev-infra`)

The details can be found in the outputs from the bootstrap.


<p>&nbsp;</p>

Each workflow references the required GitHub Secrets, so the variable as the same format as the workflow:

SERVICE_ACCOUNT = ${{ secrets.SERVICE_ACCOUNT }}

<p>&nbsp;</p>

### Manually Trigger The Workflow

<p>&nbsp;</p>

The workflow includes `on: workflow_dispatch:` which provides a manual trigger for first release.

On the GitHub main page, locate `actions`

Select the workflow `cd-workflow`

Select `Run workflow`

After the inital run, all further changes should be commited through a Pull Request/Merge into `main` to trigger the workflow.

 <p>&nbsp;</p>


### Environment Preparation

The workflow installs the following tools in order to run the CD

<p>&nbsp;</p>

For authentication into Google Cloud the GitHub actions workflow uses `google-github-actions/auth@v0`

 https://github.com/google-github-actions/auth
 

<p>&nbsp;</p>

| Purpose  | Information |
| ------------- | ------------- |
| Authenticate with Google Cloud | https://github.com/google-github-actions/auth|
| Access terrafrom state file  | https://github.com/dflook/terraform-remote-state |
| Use gcloud commands  | https://github.com/google-github-actions/setup-gcloud |
| To build the application | gcr.io/buildpacks/builder |
| Install `go` | https://github.com/actions/setup-go |
| Build Kritis Signer | https://github.com/grafeas/kritis.git |
| Deploy Google Cloud Run | https://github.com/google-github-actions/deploy-cloudrun |


<p>&nbsp;</p>

The application has flags and environment variables declared in the Cloud Run Deployment.
These are either pulled from the bootstrap state file outputs or declared manually. 

```bash
flags: |
--port=3000
--binary-authorization=default
--allow-unauthenticated
--ingress=internal-and-cloud-load-balancing
--cpu=1
--memory=512Mi
--min-instances=1
--max-instances=2
--service-account=${{ steps.tf-outputs.outputs.CLOUD_RUN_SA_EMAIL }}
```

``` bash
project_id: ${{ steps.tf-outputs.outputs.RESOURCE_PROJECT }}
region: ${{ steps.tf-outputs.outputs.REGION }}
service: ${{ env.RESOURCE_CLOUD_RUN }}
image: ${{ env.FULL_IMAGE_NAME }}:${{ env.IMAGE_TAG }}
```


# Cloud Run "Hello" container

This repository contains the source code of a sample Go application that is
distributed as the public container image (`gcr.io/cloudrun/hello`) used in the
[Cloud Run quickstart](https://cloud.google.com/run/docs/quickstarts/) and as
the suggested container image  in the Cloud Run UI on Cloud Console.

It also contains the source code of a placeholder public container
(`gcr.io/cloudrun/placeholder`)  used to create a placeholder revision when setting up 
Continuous Deployment.

[![Run on Google Cloud](https://deploy.cloud.run/button.svg)](https://deploy.cloud.run)
