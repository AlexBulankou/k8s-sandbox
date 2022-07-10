# k8s-sandbox

Start each sandbox example by creating a new project
```
gcloud auth login

export PROJECT_ID=alexbu-test-20220709
export BILLING_ACCOUNT=019970-D6BDB5-6AF850
export FOLDER_ID=52733342542

gcloud projects create $PROJECT_ID --name=$PROJECT_ID --folder=$FOLDER_ID
gcloud alpha billing projects link $PROJECT_ID --billing-account $BILLING_ACCOUNT
gcloud config set project $PROJECT_ID
```