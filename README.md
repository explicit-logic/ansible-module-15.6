# Module 15 - Configuration Management with Ansible

This repository contains a demo project created as part of my **DevOps studies** in the [TechWorld with Nana – DevOps Bootcamp](https://www.techworld-with-nana.com/devops-bootcamp).

**Demo Project:** Automate Kubernetes Deployment

**Technologies used:** Ansible, Terraform, Kubernetes, AWS EKS, Python, Linux

**Project Description:**

- Create EKS cluster with Terraform
- Write Ansible Play to deploy application in a new K8s namespace

---

## Prerequisites

Copy and fill in the variables file:

```sh
cp terraform.tfvars.example terraform.tfvars
```

Edit `terraform.tfvars` to set your AWS region and desired CIDR blocks.

---

Overview
![](./images/overview.png)

### Create EKS cluster with Terraform

Run
```sh
terraform init
terraform apply
```

![](./images/myapp-cluster.png)


### Create a Namespace in EKS cluster

Create a kubeconfig file
```sh
aws eks update-kubeconfig --region eu-central-1 --name myapp-eks-cluster --kubeconfig ./kubeconfig-myapp-eks-cluster
```

Create `deploy-to-k8s.yaml` file

```yaml
---
- name: Deploy app in new namespace
  hosts: localhost
  tasks:
    - name: Create a k8s namespace
      kubernetes.core.k8s:
        name: my-app
        api_version: v1
        kind: Namespace
        state: present
        kubeconfig: ./kubeconfig-myapp-eks-cluster
```

Doc: https://docs.ansible.com/projects/ansible/latest/collections/kubernetes/core/k8s_module.html

Install required modules:

```sh
uv sync
```

Check modules are installed

```sh
python3 -c "import yaml"
python3 -c "import kubernetes"
python3 -c "import jsonpatch"
python3 -c "import pyyaml"
```

Create `ansible.cfg`

```conf
[defaults]
host_key_checking = False
inventory = hosts
enable_plugins = aws_ec2
remote_user = ec2-user
private_key_file = ~/.ssh/id_rsa
```

Execute the playbook

```sh
ansible-playbook deploy-to-k8s.yaml
```

![](./images/execute-playbook.png)

Connect to k8s cluster

```sh
export KUBECONFIG=./kubeconfig-myapp-eks-cluster
kubectl get ns
```

![](./images/kubectl-get-ns.png)

### Deploy app in new namespace


Add deploy app step to `deploy-to-k8s.yaml`:
```yaml
    - name: Deploy nginx app
      kubernetes.core.k8s:
        src: ./nginx.yaml
        state: present
        kubeconfig: ./kubeconfig-myapp-eks-cluster
        namespace: my-app
```

Execute the playbook

```sh
ansible-playbook deploy-to-k8s.yaml
```

Check the app
```sh
kubectl get pod -n my-app
kubectl get svc -n my-app
```
![](./images/kubectl-get-app.png)

Copy `EXTERNAL-IP` and open in the browser

![](./images/nginx-app.png)

### Set environment variable for kubeconfig

```sh
export K8S_AUTH_KUBECONFIG=./kubeconfig-myapp-eks-cluster
```

Remove `kubeconfig` params from `deploy-to-k8s.yaml`

Execute the playbook again

```sh
ansible-playbook deploy-to-k8s.yaml
```

![](./images/env-var.png)

### Clean-up

Destroy kubernetes cluster

```sh
terraform destroy
```
