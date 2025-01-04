# zoo-argo-wf-workflow-template

This repository provides a Helm chart for deploying an Argo Workflow WorkflowTemplate designed as an **Application Package runner** using Calrissian. 

The WorkflowTemplate is intended to be deployed on a Kubernetes cluster with Argo Workflows installed, enabling seamless orchestration and execution of workflows defined in the **Common Workflow Language (CWL)** format. By leveraging Calrissian, this template allows efficient and scalable execution of CWL-based workflows in a containerized environment.

As an example of "business logic," this WorkflowTemplate can be customized by Platform Operators to meet specific operational requirements and use cases. It serves as a foundation for integrating CWL workflows into Kubernetes environments, providing flexibility, scalability, and robust orchestration capabilities.

## Features

- Executes CWL workflows in a Kubernetes environment using the Calrissian runner.
- Provides robust logging and error handling mechanisms.
- Supports dynamic resource allocation (RAM and CPU).
- Ensures synchronization to avoid overloading the cluster.
- Outputs workflow artifacts and logs to predefined locations.

## Parameters

The template includes the following configurable parameters:

| Parameter Name | Description                          | Default Value |
|----------------|--------------------------------------|---------------|
| `parameters`   | Parameters in JSON format           | `""`         |
| `cwl`          | CWL document in JSON format         | `""`         |
| `max_ram`      | Maximum RAM allocation (e.g., `8G`) | `8G`          |
| `max_cores`    | Maximum CPU cores                   | `8`           |
| `stac_catalog` | Location of STAC catalog JSON       | `""`         |

Ensure these parameters are appropriately configured for your CWL workflow and cluster capacity.

## Components

### Synchronization

Workflow throttling is achieved through the use of a semaphore mechanism to manage the number of concurrent workflows.

- **ConfigMap**: `semaphore-argo-cwl-runner-stage-in-out`
- **Key**: `workflow`

### Volumes

This workflow uses the following volumes:

| Volume Name                | Type                  | Purpose                                   |
|----------------------------|-----------------------|-------------------------------------------|
| `usersettings-vol`         | Secret               | Stores user settings                     |
| `calrissian-wdir`          | PersistentVolumeClaim| Working directory for Calrissian         |
| `cwl-wrapper-config-vol`   | ConfigMap            | Contains CWL wrapper configuration files |

## Workflow Templates

### 1. `calrissian-runner`

This is the entry point for executing the CWL workflow. It:

- Prepares CWL and input files.
- Executes the Calrissian job.
- Retrieves and processes the results.

### 2. `cwl-prepare`

Prepares the CWL input files by:

- Staging input and output configurations.
- Wrapping the CWL with user-defined rules and settings.

### 3. `calrissian-tmpl`

Creates and runs a Kubernetes Job to execute the workflow using Calrissian.

### 4.2. `get-results`

Handles successful workflow executions by:

- Reading output files.
- Extracting key artifacts and logs.
- Publishing results to the STAC catalog if `stac_catalog` is provided.

### 4.1. `get-results-on-failure`

Manages failures by:

- Collecting error logs and diagnostic information.
- Ensuring null artifacts for consistent outputs.
- Optionally logging failure details in the STAC catalog.

### 5. `feature-collection`

Processes the `stac-catalog` to generate a feature collection, as needed.

## Deployment Guide

To deploy this workflow template, you must have an active Argo Workflows environment running in your Kubernetes cluster. Follow these steps to successfully deploy and run the template.

**Prerequisites**:
* A Kubernetes cluster with Argo Workflows installed.
* helm package manager installed.
* kubectl configured to access the cluster.

#### Option 1: Deploy using Skaffold

If you have Skaffold installed, the simplest way to deploy the workflow template is by using the following command:
```bash
skaffold dev
```
This command automatically deploys the template and watches for changes in your code. If you don’t have Skaffold installed, follow the instructions on [Skaffold's installation page](https://skaffold.dev/docs/install/) to get it up and running.
#### Option 2: Deploy using Helm

If you don't use Skaffold, you can deploy the Helm chart directly using helm in the namespace where your Argo Workflows instance is running. 

## Additional Notes

- Ensure your Kubernetes cluster has sufficient resources to execute the CWL workflows.
- Modify the CWL wrapper script or the Calrissian image version (`ghcr.io/duke-gcb/calrissian:0.16.0`) to suit specific requirements.
- The workflow handles failures gracefully and provides diagnostic outputs to facilitate troubleshooting.
- This WorkflowTemplate is used by Zoo's Argo Workflows runner (see https://github.com/EOEPCA/zoo-argowf-runner and https://github.com/EOEPCA/zoo-argo-wf-proc-service-template)
