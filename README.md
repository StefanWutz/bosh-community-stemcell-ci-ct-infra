# BOSH Community Stemcell CI Infra

## Requirements
- gcloud => 337.0.0



## Deploy control-tower

-> one time initial tasks
Create iam user for control-tower
```
gcloud iam service-accounts create poc-controltower
````

Add iam binding to project
```
PROJECT_ID=$(gcloud config get-value core/project 2>/dev/null)
gcloud projects add-iam-policy-binding $PROJECT_ID --member="serviceAccount:poc-controltower@$PROJECT_ID.iam.gserviceaccount.com" --role="roles/owner"
```

Get gcp keys files for control-tower
```
PROJECT_ID=$(gcloud config get-value core/project 2>/dev/null)
gcloud iam service-accounts keys create gcp_iam.json --iam-account=poc-controltower@$PROJECT_ID.iam.gserviceaccount.com
export  gcp_iam.json
```

Enable api services

```
gcloud services enable compute.googleapis.com
gcloud services enable iam.googleapis.com
gcloud services enable cloudresourcemanager.googleapis.com
gcloud services enable sqladmin.googleapis.com
```


-> Deployment and upgrades
On macos
```
VERSION=0.16.16
wget https://github.com/EngineerBetter/control-tower/releases/download/$VERSION/control-tower-darwin-amd64 -O control-tower
chmod +x control-tower
```

Execute control-tower
```
PROJECT_NAME=ct-stemcell \
PROJECT_ID=$(gcloud config get-value core/project 2>/dev/null) \
GOOGLE_APPLICATION_CREDENTIALS=$(pwd)/gcp_iam.json \
GITHUB_AUTH_CLIENT_ID=<client_id> \
GITHUB_AUTH_CLIENT_SECRET=<client_secret> \
WORKERS=3 \
AWS_REGION=europe-west4 \
  ./control-tower deploy --iaas gcp ct-stemcell
```


## Destroy the control-tower
```
 GOOGLE_APPLICATION_CREDENTIALS=$(pwd)/gcp_iam.json ./control-tower destroy --iaas gcp ct-stemcell
```