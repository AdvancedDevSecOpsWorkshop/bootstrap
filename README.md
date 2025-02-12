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
2. Create a new organization called `workshop_admin`
3. In the new organization, click on Applications in the left side bar
4. Create a new app called `quay-config` and hit enter
5. Generate a token, select all permissions
6. Copy the token that is shown and create the secret with the token as follows:

```
oc create secret -n quay-operator generic api-token --from-literal=token=<token>
```

7. Go to the Cluster Argo CD and navigate to the quay-operator application
8. If the quay-operator is already syncing in Argo CD then terminate it
9. Press the Sync button again, this will restart the `configure-quay` job with the API token you provided. Note that this job is idempotent and can be run as many times as needed.

#### Gitea

The connection to Keycloak must be manually done as there doesn't appear to be an API for it. Login in as gitea-admin/openshift and then click on the icon in the
top right and select "Site Administration". Configure a new OAuth2 provider with the settings in screenshot screenshot below:

* Authentication Type: `OAuth2`
* Authentication Name: `Keycloak`
* Authentication Provider: `OpenID Connect`
* Client ID: `gitea`
* Client Secret: `c7eae76c-bd26-4588-90b5-fa7a7520fb6b`
* Icon URL: 'https://upload.wikimedia.org/wikipedia/commons/2/29/Keycloak_Logo.png'
* OpenID Connect Discovery URL: Get this from the Keycloak UI, OpenShift Realm, Realm Settings, General Tab copy the link called `OpenID Endpoint Configuration`. Alternatively the URL is `https://keycloak.<subdomain>/realms/openshift/.well-known/openid-configuration` and replace `<subdomain>` with the `apps.xxxx` subdomain for your cluster.
* Claim name providing group names for this source. (Optional): `groups`
* Group Claim value for administrator users. (Optional - requires claim name above): `cluster-admins`

![alt text](https://raw.githubusercontent.com/AdvancedDevSecOpsWorkshop/bootstrap/main/docs/img/gitea-keycloak.png)

Note when logging in with Gitea using Keycloak for the userX accounts you need to select the tab for `Link Account` since the users have been pre-created by the operator. The password to use to link the the account is `openshift`. This only needs to be done once.

#### ACS

You will need to login as the cluster administrator and under compliance press the `Scan Environment` button in order for compliance results to appear. Currently investigating how to automate this.

### Troubleshooting
#### Check in Cluster Argo CD that all Applications are in Sync and not Degraded
Login into the Cluster Argo CD and ensure that all Applications are Healthy and in Sync. Note that if Keycloak is down you may need to login using the admin account and the password stored in the openshift-gitops-cluster secret.

#### Keycloak login option doesn't appear
This most often occurs because Argo CD did not properly sync all of the applications. Login using the RHDP `admin` credential and then login into Argo CD using it's hardcoded admin/password in the `openshift-gitops-cluster` secret
as per above. In most cases fixing the issue is simply a matter of syncing whichever application is blocking the sync wave.

##### Certificate fails to be provisioned
If any of the certificates fail to be provisioned you can use the cmctl tool to request a manual renewal:

cmctl renew <xxxxx> -n <namespace>
Note you may need to restart related pods once the certificate is properly provisioned.
