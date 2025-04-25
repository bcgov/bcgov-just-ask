
# bcgov-just-ask 
[![Lifecycle:End-of-life](https://img.shields.io/badge/Lifecycle-End%20of%20life-orange)](https://github.com/bcgov/bcgov-just-ask/)

This project is end-of-life and will be archived in the future.


This is the infrastructure code for the [just-ask](https://github.com/bcgov/bcgov-just-ask/) deployment.

## How to Deploy

This app is managed by github actions. On Merge to:

- `main` an action will redeploy the infrastructure code using an ansible playbook to __production__.
- `test` an action will redeploy the infrastructure code using an ansible playbook to __test__.

It  assumes 2 environments, test and prod, with the following secrets:
* `OC_NAMESPACE` (Openshift Namespace) is set in the secrets for each environment
* `OC_TOKEN` - `pipeline-token-...` from OC Workloads > Secrets. Set in GitHub secrets for each environment.



### Deployment

Make a PR and apply the 'merge when ready' label when everythings approved! The PR will auto merge and a github action will take care of the rest. 


