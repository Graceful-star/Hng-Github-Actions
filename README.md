# Hng-Github-Actions
# Self-Hosted GitHub Actions Runners on Kubernetes - Documentation

## Table of Contents

1. [Introduction](#introduction)
2. [Prerequisites](#prerequisites)
3. [Setting Up the GitHub Organization and Repository](#setting-up-the-github-organization-and-repository)
4. [Setting Up a Kubernetes Cluster with Minikube](#setting-up-a-kubernetes-cluster-with-minikube)
5. [Installing the GitHub Actions Runner Controller](#installing-the-github-actions-runner-controller)
6. [Deploying the GitHub Actions Runners](#deploying-the-github-actions-runners)
7. [Testing the Setup](#testing-the-setup)
8. [Documentation and Deliverables](#documentation-and-deliverables)
9. [Conclusion](#conclusion)

## Introduction

This documentation will guide you through the process of setting up self-hosted GitHub Actions runners on a Kubernetes cluster using the GitHub Actions Runner Controller. By the end of this guide, you will have a functioning CI/CD pipeline that leverages Kubernetes to manage your GitHub Actions runners. This setup is scalable and ensures that your runners can handle multiple workflows efficiently.

## Prerequisites

Before starting, ensure you have the following:

- A GitHub account.
- Basic understanding of Git and GitHub.
- A machine with Docker installed (necessary for Minikube).
- Administrative access to your system (for installing tools and managing Docker).

## Setting Up the GitHub Organization and Repository

### 1. Create a GitHub Organization

- Go to GitHub and create a new organization. This will allow you to manage repositories under a common account.
- Name your organization appropriately (e.g., `my-org`).

### 2. Fork a Repository

- Fork one of the HNG boilerplate repositories into your newly created organization. This repository will be used for testing the self-hosted runners.

### 3. Clone the Repository

- Clone the forked repository to your local machine to work on it.

```bash
git clone https://github.com/your-org/your-repo.git
cd your-repo

```

## Setting Up a Kubernetes Cluster with Minikube

### 1. Install Minikube

- Install Minikube on your local machine. Minikube is a tool that lets you run a Kubernetes cluster locally.

  - For Linux:

    ```bash
    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    sudo install minikube-linux-amd64 /usr/local/bin/minikube
    ```

  - For Windows and macOS, follow the instructions on the [Minikube installation page](https://minikube.sigs.k8s.io/docs/start/).

### 2. Start Minikube

- Start Minikube using the Docker driver.

    ```bash
    minikube start --driver=docker
    ```

### 3. Troubleshooting

- If you encounter permission issues with Docker, add your user to the Docker group:

    ```bash
    sudo usermod -aG docker $USER
    newgrp docker
    ```

- Restart Minikube after resolving any issues.

## Installing the GitHub Actions Runner Controller

### 1. Create a Dedicated Namespace

- In your Kubernetes cluster, create a dedicated namespace for the GitHub Actions Runner Controller.

    ```bash
    kubectl create namespace actions-runner-system
    ```

### 2. Install Helm

- Install Helm, a package manager for Kubernetes:

  - For Linux:

    ```bash
    curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
    ```

  - For Windows and macOS, follow the instructions on the [Helm installation page](https://helm.sh/docs/intro/install/).

### 3. Install the GitHub Actions Runner Controller

- Add the Helm repository and install the controller:

    ```bash
    helm repo add actions-runner-controller https://actions-runner-controller.github.io/actions-runner-controller
    helm repo update
    helm install -n actions-runner-system actions-runner-controller actions-runner-controller/actions-runner-controller
    ```

## Deploying the GitHub Actions Runners

### 1. Create a Runner Deployment YAML

- Create a `runners.yaml` file that will define the deployment of the GitHub Actions runners:

    ```yaml
    apiVersion: actions.summerwind.dev/v1alpha1
    kind: RunnerDeployment
    metadata:
      name: example-runner-deployment
      namespace: actions-runner-system
    spec:
      replicas: 2
      template:
        spec:
          repository: your-org/your-repo
          labels:
            - self-hosted
            - linux
          token: your-github-token
    ```

### 2. Apply the YAML

- Apply the `runners.yaml` file to deploy the runners:

    ```bash
    kubectl apply -f runners.yaml
    ```

### 3. Verify the Deployment

- Verify that the runners are deployed and registered with your GitHub repository:

    ```bash
    kubectl get runners -n actions-runner-system
    ```

## Testing the Setup

### 1. Create a GitHub Actions Workflow

- Create a `.github/workflows/test.yml` file in your repository to define a simple GitHub Actions workflow:

    ```yaml
    name: Test Self-Hosted Runners

    on: [push]

    jobs:
      build:
        runs-on: self-hosted
        steps:
        - name: Check out code
          uses: actions/checkout@v2
        - name: Run a one-liner script
          run: echo "This is running on a self-hosted runner!"
    ```

### 2. Push the Workflow

- Push the workflow to your GitHub repository:

    ```bash
    git add .
    git commit -m "Add test workflow"
    git push
    ```

### 3. Monitor the Workflow

- Monitor the workflow in GitHub to ensure it runs on your self-hosted runners. You should see the workflow run under the "Actions" tab of your repository.



## Conclusion

This documentation has walked you through the process of setting up self-hosted GitHub Actions runners on a Kubernetes cluster using Minikube and the GitHub Actions Runner Controller. Following these steps will enable you to create scalable and manageable CI/CD pipelines that leverage Kubernetes for efficient workflow management.

