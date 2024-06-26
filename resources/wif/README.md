# How WIF templates work - high level

When a customer creates an OSD-GCP cluster using Workload Identity Federation,
a wif_config object is made. The content of that wif_config object lists the
service accounts and permissions needed for the cluster's deployment and
operation. This criteria is defined here, and then passed to OCM where it hosts
an API endpoint for clients to query. The API provided by Clusters Service
watches for changes in [this](https://gitlab.cee.redhat.com/service/app-interface/-/blob/master/data/services/ocm/shared-resources/common.yml)
file. When this file changes a new [ConfigMap](https://gitlab.cee.redhat.com/service/app-interface/-/blob/master/resources/services/ocm/gcp-wif-template.configmap.yaml)
is templated which Clusters Service pods (OCM) will read.

## How to update WIF templates

When a new WIF template needs to be added or an existing one changed, follow this process:

1. Create a PR to this repo with the desired changes to the policies. 
2. Once merged, Create an MR to [Cluster Services](https://gitlab.cee.redhat.com/service/app-interface/-/blob/master/data/services/ocm/shared-resources/common.yml) to update the referenced commit hash to consume new changes. 
3. Cluster service will restart pods automatically making policy changes available almost immediately.

## Example wif_template Scheme

```yaml
id: v4.14
kind: WifTemplate
service_accounts:
  - id: osd-deployer
    kind: ServiceAccount
    osd_role: deployer
    access_method: impersonate
    roles:
      - id: compute.admin
        kind: Role
        predefined: true
      - id: dns.admin
        kind: Role
        predefined: true
  - access_method: wif
    credential_request:
      SecretRef:
        Name: cloud-credential-operator-gcp-ro-creds
        Namespace: openshift-cloud-credential-operator
      ServiceAccountNames:
        - cloud-credential-operator
    id: cloud-credential-operator-gcp-ro-creds
    kind: ServiceAccount
    osd_role: cloud-credential-operator-gcp-ro-creds
    roles:
      - id: iam.securityReviewer
        kind: Role
        predefined: true
      - id: iam.roleViewer
        kind: Role
        predefined: true
service_apis:
  - deploymentmanager.googleapis.com
  - compute.googleapis.com
```
