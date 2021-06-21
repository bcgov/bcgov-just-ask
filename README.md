
# bcgov-just-ask 
[![Lifecycle:Stable](https://img.shields.io/badge/Lifecycle-Stable-97ca00)](https://github.com/bcgov/bcgov-just-ask/)


This is the infrastructure code for a [just-ask](https://justask.cloud) deployment to manage github org membership into bcgov/bcdevops

## How to Deploy

This app is managed by github actions. On Merge to:

- `main` an action will redeploy the infrastructure code using an ansible playbook to __production__.
- `test` an action will redeploy the infrastructure code using an ansible playbook to __test__.

### Pre Requisites
1. Login to Openshift
2. Adjust the group vars to your liking
3. Create the Service Account and grab the credentials to stuff inside the repo secrets
`ansible-playbook create-sa.yaml`
4. Grab the service account secrets token and stuff into a github secret called `OPENSHIFT_SA_PASSWORD`
` oc get secret just-ask-token-<id> -o json | jq -r '.data.token' | base64 -D'
5. Grab Openshift server url and stuff into a github secret called `OPENSHIFT_SERVER_URL`

There are several secrets you will need to grab from the Github App that has been installed on Github. Things like the __private key__, __client secret__, __client id__, and __app_id__. Obtain those and store as github secret as such:

- PRIVATE_KEY: github app private key (.pem file) __base64 encode the contents before saving__
- CLIENT_ID: github app client id
- CLIENT_SECRET: github app client secret
- APP_ID: github app app id

There are also secrets needed for the database. 

- DB_ADMIN_PASSWORD
- DB_PASSWORD
- DB_NAME
- DB_USER

### Deployment

Make a PR and apply the 'merge when ready' label when everythings approved! The PR will auto merge and a github action will take care of the rest. 


### Setting up DB Backups

Database backups on OCP 4.x is done with the [`backup-container`](https://github.com/bcdevops/backup-container). Setup is straight forward. 

1. Create the `backup.conf` configmap by running the `backup-container` playbook and then apply in the __prod namespace__
2. As per the instructions for deploying backup container. Create a new build for the backup container in the __tools namespace__:

```
curl https://raw.githubusercontent.com/BCDevOps/backup-container/master/openshift/templates/backup/backup-build.yaml |
oc process -f - -p NAME=just-ask-backup -p DOCKER_FILE_PATH=Dockerfile_Mongo -p OUTPUT_IMAGE_TAG=1.0.0 -p BASE_IMAGE_FOR_BUILD=registry.access.redhat.com/rhscl/mongodb-36-rhel7 | oc apply -f -
```

3. As per instructions, deploy the backup container by first modifying the templates env vars then applying in the __prod namespace__

```

curl https://raw.githubusercontent.com/BCDevOps/backup-container/master/openshift/templates/backup/backup-deploy.yaml | sed -e 's/DATABASE_USER$/JUST_ASK_DB_USER/; s/DATABASE_PASSWORD$/JUST_ASK_DB_PASSWORD/' | oc process -f - -p NAME=just-ask-backup -p SOURCE_IMAGE_NAME=just-ask-backup -p APP_NAME=just-ask -p CPU_LIMIT=1000m -p MEMORY_LIMIT=1Gi -p NAMESPACE_NAME=<prod namespace> -p CPU_REQUEST=300m -p MEMORY_REQUEST=500Mi -p ENVIRONMENT_NAME=prod -p IMAGE_NAMESPACE=<tools namespace> -p TAG_NAME=1.0.0 -p DATABASE_DEPLOYMENT_NAME=just-ask-db | oc apply -f -
```