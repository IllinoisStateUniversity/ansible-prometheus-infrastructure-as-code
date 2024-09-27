# ansible-prometheus-infrastructure-as-code

## Overview

This Ansible playbook automates the deployment and configuration of Prometheus instances in a Kubernetes or OpenShift cluster using Infrastructure as Code (IaC) principles. It is designed to run within AAP/AWX and supports multiple environments (e.g., production, test) through environment-specific variable files.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Playbook Structure](#playbook-structure)
- [Variables](#variables)
  - [Environment Selection](#environment-selection)
  - [Variable Files](#variable-files)
- [Usage](#usage)
  - [Running the Playbook](#running-the-playbook)
- [Customization](#customization)
- [Dependencies](#dependencies)
- [Troubleshooting](#troubleshooting)
- [Contact](#contact)

## Prerequisites

Before running this playbook, ensure you have the following:

- **AAP/AWX**: The playbook is designed to run within AAP/AWX.
- **Kubernetes/OpenShift Access**: Credentials to interact with your cluster.
- **Keeper Secrets Manager Access**: For retrieving secrets like bearer tokens.

## Playbook Structure

- **`site.yml`**: Main playbook file.
- **`vars/`**: Directory containing environment-specific variable files (`prod.yml`, `test.yml`).
- **`roles/prometheus/`**: Role responsible for deploying Prometheus instances.
  - **`tasks/main.yml`**: Tasks to create ConfigMaps, Deployments, and Services.
  - **`templates/`**: Jinja2 templates for Kubernetes manifests.
- **`collections/requirements.yml`**: Lists required Ansible collections.

## Variables

### Environment Selection

Set the `prometheus_environment` variable to choose the target environment:

- **AAP/AWX**:

  In the **Extra Variables** field of your job template:

  ```yaml
  prometheus_environment: prod
  ```

### Variable Files

Located in the `vars/` directory:

- **`prod.yml`**: Variables for the production environment.
- **`test.yml`**: Variables for the test environment.

Each file defines `prometheus_instances`, a list of instances to deploy and their strape configs.

#### Example (`vars/prod.yml`):

```yaml
prometheus_instances:
  - instance_number: '01'
    namespace: metrics
    mimir_url: http://mimir-service.metrics.svc.cluster.local:9009/api/v1/push
    scrape_interval: 60s
    evaluation_interval: 60s
    rule_files: |
      rule_files:
        # - "first.rules"
        # - "second.rules"
    scrape_configs: |
      scrape_configs:
        - job_name: prometheus-01
          honor_labels: true
          static_configs:
            - targets: ['localhost:9090']
        # Additional scrape configs...
```

## Usage

### Running the Playbook

1. **Configure Variables**:

   - Ensure the appropriate variable file exists in `vars/`.
   - Set `prometheus_environment` to match the desired environment.

3. **Set Up AAP/AWX Job Template**:

   - **Inventory**: Use `localhost` or an inventory containing `localhost`.
   - **Credentials**: Add credentials for Kubernetes/OpenShift and Keeper Secrets Manager.
   - **Extra Variables**: Set `prometheus_environment`.

4. **Run the Job**:

   - Launch the job from AWX/Ansible Tower.
   - Monitor the job output for any errors.

## Customization

- **Namespaces**: Modify `namespace` in the variable files to change where Prometheus is deployed.
- **Scrape Configurations**: Update `scrape_configs` to adjust which endpoints Prometheus monitors.
- **Rule Files**: Define custom Prometheus rules in `rule_files`.

## Dependencies

- **Ansible Collections**:

  - `redhat.openshift`
  - `kubernetes.core`
  - `keepersecurity.keeper_secrets_manager`

- **Keeper Secrets Manager**: Ensure access is configured for secret retrieval.
- **Kubernetes/OpenShift Cluster**: Accessible from the Ansible execution environment.

## Troubleshooting

- **Authentication Failures**:

  - Verify Kubernetes/OpenShift credentials.
  - Check Keeper Secrets Manager configuration.

- **Resource Creation Errors**:

  - Ensure the target namespaces exist or adjust the playbook to create them.
  - Check for typos or misconfigurations in variable files.

- **Collection Issues**:

  - Ensure all required collections are installed.
  - Update collections if compatibility issues arise.
