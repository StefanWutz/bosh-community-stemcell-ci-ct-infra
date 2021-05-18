# BOSH Community Stemcell CI Infra

Proof of Concept to install [control-tower](https://github.com/EngineerBetter/control-tower) on Google Cloud.


## Requirements
The following command line tools should be installed to continue

- gcloud
- credhub client
- fly


## Deploy control-tower on gcp
Login to gcp using glcoud

```
gcloud auth login
```

The project to deploy is _cloud-foundry-310819_. It should be mentioned in the info section after you successfully logged in, otherwise switch to the project

```
gcloud config set project _cloud-foundry-310819
```

### Initial tasks (only needs be created once)
- Create iam user for control-tower
```
gcloud iam service-accounts create poc-controltower
```

- Add iam binding to project
```
PROJECT_ID=$(gcloud config get-value core/project 2>/dev/null)
gcloud projects add-iam-policy-binding $PROJECT_ID --member="serviceAccount:poc-controltower@$PROJECT_ID.iam.gserviceaccount.com" --role="roles/owner"
```

- Get gcp keys files for control-tower
```
PROJECT_ID=$(gcloud config get-value core/project 2>/dev/null)
gcloud iam service-accounts keys create gcp_iam.json --iam-account=poc-controltower@$PROJECT_ID.iam.gserviceaccount.com
```

- Export gcp file location for further use
```
export GOOGLE_APPLICATION_CREDENTIALS=$(pwd)/gcp_iam.json
```

- Enable api services

```
gcloud services enable compute.googleapis.com
gcloud services enable iam.googleapis.com
gcloud services enable cloudresourcemanager.googleapis.com
gcloud services enable sqladmin.googleapis.com
```


### Deployment and upgrades
On macos
```
VERSION=0.16.17
wget https://github.com/EngineerBetter/control-tower/releases/download/$VERSION/control-tower-darwin-amd64 -O control-tower
chmod +x control-tower
```

Execute control-tower
```
PROJECT_ID=$(gcloud config get-value core/project 2>/dev/null) \
GOOGLE_APPLICATION_CREDENTIALS=$(pwd)/gcp_iam.json \
GITHUB_AUTH_CLIENT_ID=<client_id> \
GITHUB_AUTH_CLIENT_SECRET=<client_secret> \
WEB_SIZE=medium \
WORKERS=3 \
WORKER_SIZE=xlarge \
DOMAIN=ct-stemcell-poc.ci.cloudfoundry.org \
AWS_REGION=europe-west4 \
  ./control-tower deploy --iaas gcp ct-stemcell
```


## Access cluster resources
Use the command
```
GOOGLE_APPLICATION_CREDENTIALS=$(pwd)/gcp_iam.json  ./control-tower info --iaas gcp ct-stemcell
```

to get all information necessary to access the cluster resources, concourse, credhub and grafana.

## Destroy the control-tower
```
GOOGLE_APPLICATION_CREDENTIALS=$(pwd)/gcp_iam.json ./control-tower destroy --iaas gcp ct-stemcell
```

# Upgrades and changing configuration

## Upgrade existing control-tower platform
To upgrade to the latest version, download the latest version of control-tower binary. Execute `control-tower` as defined in the previous section. The underlying BOSH infrastructure will update all vms.

## Changing configuration
Changing executing configuration means to adopt the existing `control-tower` properties and pass new values during a deployment, see previous section. Some properties of concourse might not be exposed as properties, therefore you have to find alternative options to apply the changes.
For example managing teams and their authentication settings by using [Concourse Management](https://github.com/EngineerBetter/concourse-mgmt).


# Domain
A custom domain for concourse is supported, see https://github.com/EngineerBetter/control-tower/blob/master/docs/deploy.md#custom-domains. A hosted zone `ci-cloudfoundry-org` for concourse already exists in the cloudfoundry gcp project.
The domain is set by adding the _DOMAIN_ property to the deploy command:
```
DOMAIN=ct-stemcell-poc.ci.cloudfoundry.org \
```

This will create a new NS entry in the gcp dns hosted zone `ci-cloudfoundry-org`.


# Pro/Cons List

## Pros
- Easy setup on an existing gcp project. One-liner command containing all required values to set up a concourse cluster
- Upgrading the cluster is the same procedures (commands) as to create a cluster. Just download the latest control-tower release
- Grafana monitoring dashboard works out of the box
- Self upgrade pipeline



## Cons
- There is no auto-scaling functionality for BOSH or Concourse available. If worker nodes have to been scaled then a new control-tower with updated number of worker nodes have to be executed
- Only one worker instance type per deployment
- Main Github Team has to be set after each cluster upgrade
- No all parameters are exposed to the command line tool. Some settings to concourse or other deployments might be harder to solve
- Deployment name can have only 11 characters


# TODOs
- Add real main github team for authentication
- Add security group/source range to access resources from SAP network to access control-tower metadata
- More Grafana dashboards for BOSH, credhub, grafana


