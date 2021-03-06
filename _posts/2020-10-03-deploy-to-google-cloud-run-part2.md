---
layout: post
published: true
disqus: true
fbcomments: false
title: Automated deployment to Google Cloud Run
---
## Checklist
- [Google Cloud Account](https://cloud.google.com/free)[https://cloud.google.com/iam/docs/creating-managing-service-account-keys](https://cloud.google.com/iam/docs/creating-managing-service-account-keys)
- [Google Cloud CLI](https://cloud.google.com/sdk/docs/install)
- Dockerized Application, [see part 1 of this article](https://deenar.github.io./dockerising-scala-app-with-sbt/)

## Step 1 - Create a GCP Service Account
- [Create a service account under IAM](https://cloud.google.com/iam/docs/creating-managing-service-accounts)
![Create GCP Service Account]({{site.baseurl}}/media/gcpserviceaccount.png)

- Grant it the following roles
	- Storage Admin: for pushing docker images to GCR.
	- Cloud Run Admin: for deploying a service to Cloud Run.
	- IAM Service Account user
    
![Grant IAM roles]({{site.baseurl}}/media/iamroles.png)

- [Create a key and download the credentials](https://cloud.google.com/iam/docs/creating-managing-service-account-keys)

## Step 2 - Push your Docker image to [GCR](https://cloud.google.com/container-registry)
Customise your Docker image publishing in build.sbt


	dockerBaseImage := "java:8-jre"
	packageName in Docker := "<you-gcloud-project-id>/api"
	maintainer in Docker := "Maintainer"
	packageSummary := "Package summary"
	packageDescription := "Package description"
	dockerRepository := Some("us.gcr.io")
    
- Substitute your Google Cloud Project ID in the packageName field
- Set dockerRepository to whichever gcr.io host you want to use. e.g. `us.gcr.io`
- Authenticate using the Google Cloud SDK using the service account credentials used in Step 1
- Run `sbt docker:publish`
- Verify using `gcloud container images list --repository eu.gcr.io/payment-rules`

## Step 3 - Create a Google Cloud Run service

- [Head to Google Cloudn Run and create a new service](https://console.cloud.google.com/run)
- Select a region
- Pick a service name, e.g. `api`
- Select unauthenticated invocations
- Choose the container image you uploaded to GCR in Step 2 e.g. `eu.gcr.io/payment-rules/api:1.0`
- Ask GCP to forward traffic to the port your application listens on and set any environment variables required. Alternatively your app should use the $PORT environment variable to listen on.
- Choose the service account created in Step 1 to run the application
- Under advanced setting choose memory and CPU allocation and auto scaling parameters
- Press Create and your app is ready to serve your clients

![Create Google Cloud Run Service]({{site.baseurl}}/media/createService.jpg)

 
## Step 4 - Check your application    
- GCP deploys your application and makes it available on the URL listed
- Check you application 
- Look at logs in Stackdriver for troubleshooting

    
## Step 5 - Automate the boring stuff so you can focus on the important bits    
I am using bitbucket as the repository, here is my /bitbucket-pipelines.yml. I have added a pipeline to deploy to GCP. It listens to changes on tag master release-gcp-* and kicks of the process we did manually before. I store the service credentials in bitbuckets environment variables, which are encrypted and stored in a vault.
    
```
image: deenar/scala-sbt:latest
# enable Docker for your repository
options:
  docker: true

pipelines:
  default:
    - step:
        script: 
          - sbt test
  tags:
    release-cloudrun-*:
    - step:
        script: # Build your repository and publish a docker image
          - sbt docker:publishLocal
          - docker save -o tmp-image.docker eu.gcr.io/payment-rules/api:1.0
        artifacts:
          - tmp-image.docker

    - step: # push the docker image to GCP using service acc credentials
          name: Build - Push - Deploy to GCP (gcr.io/payment-rules/api) for Production
          image: google/cloud-sdk:latest
          caches:
            - docker
          deployment: production
          script:
            - export GCLOUD_PROJECT=payment-rules
            - echo $GCLOUD_API_KEYFILE | base64 --decode --ignore-garbage> ./gcloud-api-key.json
            - gcloud auth activate-service-account --key-file gcloud-api-key.json

            # Linking to the Google Cloud project
            - gcloud config set project $GCLOUD_PROJECT          
            - gcloud config list

            # set image name
            - export IMAGE_NAME=eu.gcr.io/payment-rules/api:1.0 # ex. gcr.io/my-g-project/my-cr-service
            - export SERVICE_NAME=api

            # Gcloud auth and check
            - gcloud auth activate-service-account payment-rules@appspot.gserviceaccount.com --key-file=gcloud-api-key.json
            - gcloud config list

            # config image registry with gcloud helper
            - gcloud auth configure-docker -q

            # push image to gcr
            - docker load --input ./tmp-image.docker
            - docker push $IMAGE_NAME

            # deploy to cloud run
            - gcloud beta run deploy $SERVICE_NAME --image $IMAGE_NAME --region us-central1 --project payment-rules --platform managed --allow-unauthenticated --memory=512Mi

            # :-)
            - echo "Hope you found this blog post useful"
```

Please let me know if you found this useful.
