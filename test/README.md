# Test infrastructure

## GCS test setup

1. GCS bucket we can push random stuff to

`tekton-catalog-test-asdf`

2. Service account that can push to the bucket & Json file with credentials

```bash
SERVICE_ACCOUNT_NAME=catalog-test-gcs-account
SERVICE_ACCOUNT_DEST=catalog-test-gcs-account.json
PROJECT=tekton-releases

gcloud config set project $PROJECT

gcloud iam service-accounts create \
    $SERVICE_ACCOUNT_NAME \
    --display-name $SERVICE_ACCOUNT_NAME

SA_EMAIL=$(gcloud iam service-accounts list \
    --filter="displayName:$SERVICE_ACCOUNT_NAME" \
    --format='value(email)')

gcloud projects add-iam-policy-binding $PROJECT \
    --role roles/storage.admin --member serviceAccount:$SA_EMAIL

gcloud iam service-accounts keys create $SERVICE_ACCOUNT_DEST \
    --iam-account $SA_EMAIL
```

3. Secret that contains json file

```bash
kubectl create secret generic catalog-test-gcs --from-file=$SERVICE_ACCOUNT_DEST
```

4. PVC or emptydir if no validation

## Invoking stuff

```bash
kubectl apply -f gcs/gcs-input.yaml
kubectl apply -f gcs/samples/gcs-sample.yaml

# hit endpoint somehow
curl $(k get service el-gcs-listener -o jsonpath='{ .status.loadBalancer.ingress[0].* }:{ .spec.ports[0].port }')
```