Kubernetes the Hard Way: Ansible
================================

This set of playbooks prepares small kubernetes infrastructure in Google Cloud
Engine. All steps are based on ["Kubernetes The Hard Way"](https://github.com/kelseyhightower/kubernetes-the-hard-way/tree/1.8.0) tutorial for Kubernetes 1.8.0.

## Prerequisites

- configured Google Cloud engine account
- gcloud should be authorized to use the target project and zone as default (check: `gcloud info` should output somethin)
- service account for access gcloud instances should be created

- `git clone https://github.com/vbrednikov/kthw-ansible.git`
- `pip install -r requirements.txt`

## Configure GCE settings

See official [Google Cloud Platform Guide](http://docs.ansible.com/ansible/latest/guide_gce.html) for details on configuring access to GCE. In short, the steps are:

1. Download [gce.py](https://github.com/ansible/ansible/blob/devel/contrib/inventory/gce.py) and [gce.ini](https://github.com/ansible/ansible/blob/devel/contrib/inventory/gce.ini) from [ansible contrib inventory folder](https://github.com/ansible/ansible/blob/devel/contrib/inventory/).
2. In [Google cloud IAM service accounts](https://console.cloud.google.com/iam-admin/serviceaccounts/project) for your project, create an account named "inventory" with role "Project viewer".
  - write down Service Account ID (e.g., `inventory@infra-XXXXXX.iam.gserviceaccount.com`)
  - enable checkbox "Furnish a new private key", select "JSON" format
3. Open downloaded json file in any text editor,
  - extract value of private key (from "-----BEGIN PRIVATE KEY-----" till "-----END PRIVATE KEY-----" to new file named `~/ansible_gce/inventory.pem`
  - replace `\n` string with newlines
  - save the file
4. In `gce.ini`, you will need to edit variables:
  - `gce_service_account_email_address`: service account id
  - `gce_service_account_pem_file_path`: full path to `~/ansible_gce/inventory.pem`
  - `gce_project_id`: google cloud ID of your project
5. chmod +x ./gce.py and run it as `./gce.py --list --pretty`. If everything is done correctly, json-formatted data about your environment will be printed.
6. Create file `inventory/private-vars.yml` with the following contents (based on the same settings as in gce.ini):
```
---
all:
  vars:
    service_account_email: ansible@docker-181920.iam.gserviceaccount.com
    credentials_file: /Users/vbrednikov/otus-devops/kthw2/docker-7816912a55f9.json
    project_id: docker-181920
    region: europe-west1
    zone: europe-west1-b
    disk_size: 20
```

## Run playbooks

### Configure network and start the instances

Please note that only one file should be specified as inventory, instead the full folder.

```
ansible-playbook -i inventory/private_vars.yml 01_gce_instances.yml
```

### Prepare keys and configs (locally)

```
ansible-playbook -i inventory/ 02_local_conf.yml
```

### Provision worker and controller hosts

```
ansible-playbook -i inventory/ 03_configure_instances.yml
```

### After-party

After all these steps, local kubectl should be able to manage the k8s environment.

Run `kubectl get cs` and `kubectl get nodes` locally to check controllers and workers status.

Run some checks described on [Smoke tests](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/1.8.0/docs/13-smoke-test.md) page of the tutorial.
