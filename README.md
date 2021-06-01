
# bcgov-just-ask 
[![Lifecycle:Stable](https://img.shields.io/badge/Lifecycle-Stable-97ca00)](https://github.com/bcgov/bcgov-just-ask/)


This is the infrastructure code for a [just-ask](https://justask.cloud) deployment to manage github org membership into bcgov/bcdevops

## How to Deploy

This app is managed by github actions. On Merge to main, an action will redeploy the infrastructure code using an ansible playbook.

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


