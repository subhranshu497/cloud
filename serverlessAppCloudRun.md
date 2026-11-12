# Cloud Run Deployment Tasks

## Task 1: Staging API deployment

```bash
cd ~/pet-theory/lab07/unit-api-billing
gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/billing-staging-api:0.1
gcloud run deploy public-billing-service-456 \
  --image gcr.io/$GOOGLE_CLOUD_PROJECT/billing-staging-api:0.1 \
  --platform managed \
  --region us-east4 \
  --allow-unauthenticated
```

## Task 2: Staging frontend deployment

```bash
cd ~/pet-theory/lab07/staging-frontend-billing
gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-staging:0.1
gcloud run deploy frontend-staging-service-790 \
  --image gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-staging:0.1 \
  --platform managed \
  --region us-east4 \
  --allow-unauthenticated
```

## Task 3: Replace public billing service with a private version

```bash
gcloud run services delete public-billing-service-456 --region us-east4

cd ~/pet-theory/lab07/staging-api-billing
gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/billing-staging-api:0.2
gcloud run deploy private-billing-service-961 \
  --image gcr.io/$GOOGLE_CLOUD_PROJECT/billing-staging-api:0.2 \
  --platform managed \
  --region us-east4

BILLING_URL=$(gcloud run services describe private-billing-service-961 \
  --region us-east4 \
  --format "value(status.url)")

curl -X GET \
  -H "Authorization: $(gcloud auth print-identity-token)" \
  "$BILLING_URL"
```

## Task 4: Create and configure service account for billing service

> Permission issue occurred while creating the service account initially.

```bash
gcloud iam service-accounts create billing-service-sa-275 \
  --display-name "Billing Service Cloud Run"

gcloud iam service-accounts list

gcloud iam service-accounts update billing-service-sa-275@qwiklabs-gcp-02-353da08b48f2.iam.gserviceaccount.com \
  --display-name="Billing Service Cloud Run"

gcloud builds submit --tag gcr.io/qwiklabs-gcp-02-353da08b48f2/billing-service

gcloud run deploy billing-service \
  --image gcr.io/qwiklabs-gcp-02-353da08b48f2/billing-service \
  --platform managed \
  --region us-east4 \
  --allow-unauthenticated \
  --service-account billing-service-sa-275@qwiklabs-gcp-02-353da08b48f2.iam.gserviceaccount.com

gcloud iam service-accounts update billing-service-sa-275@qwiklabs-gcp-02-353da08b48f2.iam.gserviceaccount.com \
  --display-name="Billing Service Cloud Run"
```

## Task 5: Production API deployment

```bash
cd ~/pet-theory/lab07/prod-api-billing
gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/billing-prod-api:0.1
gcloud run deploy billing-prod-service-169 \
  --image gcr.io/$GOOGLE_CLOUD_PROJECT/billing-prod-api:0.1 \
  --platform managed \
  --region us-east4 \
  --no-allow-unauthenticated \
  --service-account billing-service-sa-275@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com

PROD_BILLING_URL=$(gcloud run services describe billing-prod-service-169 \
  --platform managed \
  --region us-east4 \
  --format "value(status.url)")

curl -X GET \
  -H "Authorization: $(gcloud auth print-identity-token)" \
  "$PROD_BILLING_URL"
```

## Task 6: Grant Cloud Run invoker access

```bash
gcloud iam service-accounts update frontend-service-sa-147@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com \
  --display-name="Billing Service Cloud Run Invoker"

gcloud run services add-iam-policy-binding billing-prod-service-169 \
  --member="serviceAccount:frontend-service-sa-147@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com" \
  --role="roles/run.invoker" \
  --region=us-east4
```

## Task 7: Production frontend deployment

```bash
cd ~/pet-theory/lab07/prod-frontend-billing
gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-prod:0.1
gcloud run deploy frontend-prod-service-269 \
  --image gcr.io/$GOOGLE_CLOUD_PROJECT/frontend-prod:0.1 \
  --platform managed \
  --region us-east4 \
  --allow-unauthenticated \
  --service-account frontend-service-sa-147@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
```

## Summary

This sequence deploys the billing and frontend services across staging and production, creates the required Cloud Run IAM service accounts, and grants the necessary run.invoker permission for private backend access.
