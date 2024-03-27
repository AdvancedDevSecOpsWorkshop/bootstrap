### Installation

Eventually this workshop will be available in RHDP but for now it can be installed as follows:

Provision OpenShift Workshop 4.15 instance in RHDP with the desired number of users

Login on the command line as the admin user with the credentials RHDP provided

Modify bootstrap/bootstrap/ansible/bootstrap.yaml to set the number of users you need, this must match the number of users selected in RHDP

Run the command ./bootstrap/bootstrap.sh

Note that the installation is using ansible to bootstrap GitOps onto the cluster so you must have ansible installed on your system along with the requirements to use the ansible k8s module.

### Troubleshooting
#### Check in Cluster Argo CD that all Applications are in Sync and not Degraded
Login into the Cluster Argo CD and ensure that all Applications are Healthy and in Sync. Note that if Keycloak is down you may need to login using the admin account and the password stored in the openshift-gitops-cluster secret.

##### Certificate fails to be provisioned
If any of the certificates fail to be provisioned you can use the cmctl tool to request a manual renewal:

cmctl renew <xxxxx> -n <namespace>
Note you may need to restart related pods once the certificate is properly provisioned.
