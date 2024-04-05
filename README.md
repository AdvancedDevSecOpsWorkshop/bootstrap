### Installation

Eventually this workshop will be available in RHDP but for now it can be installed as follows:

Provision OpenShift Workshop 4.15 instance in RHDP with the desired number of users

Login on the command line as the admin user with the credentials RHDP provided

Modify bootstrap/bootstrap/ansible/bootstrap.yaml to set the number of users you need, this must match the number of users selected in RHDP

Run the command ./bootstrap/bootstrap.sh

Note that the installation is using ansible to bootstrap GitOps onto the cluster so you must have ansible installed on your system along with the requirements to use the ansible k8s module.

### Manual Installation

While the workshop installation is automated there are a couple of items that must be manually configured.

#### Quay

The workshop uses Quay with Keycloak and unfortunately Quay does not provide any way to get an API token when using OIDC authentication. As a result
you will need to log into Quay with the `clusteradmin` user and create the Admin token.

Follow these steps:

1. Login into Quay via Kerycloak using the `clusteradmin` user and provided password
2. Create a new organization called `workshop-admin`
3. In the new org

5. Create the secret with the token as follows:

```
oc create secret generic api-token --from-literal=token=<token>
```

6. Go to the Cluster Argo CD and navigate to the quay-operator application
7. If the quay-operator is already syncing in Argo CD then terminate it
8. Press the Sync button again, this will restart the `configure-quay` job with the API token you provided. Note that this job is idempotent and can be run as many times as needed.

### Troubleshooting
#### Check in Cluster Argo CD that all Applications are in Sync and not Degraded
Login into the Cluster Argo CD and ensure that all Applications are Healthy and in Sync. Note that if Keycloak is down you may need to login using the admin account and the password stored in the openshift-gitops-cluster secret.

##### Certificate fails to be provisioned
If any of the certificates fail to be provisioned you can use the cmctl tool to request a manual renewal:

cmctl renew <xxxxx> -n <namespace>
Note you may need to restart related pods once the certificate is properly provisioned.
