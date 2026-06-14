# Github Workflow Monitor

## Overview

The **Github Workflow Monitor** is an n8n workflow designed to ensure that critical GitHub Actions workflows remain active. It periodically checks the status of a specific workflow across multiple repositories and automatically re-enables it if it has been disabled. This is particularly useful for maintaining continuous integration and deployment (CI/CD) pipelines without manual intervention.

## Prerequisites

To successfully run this workflow, you will need:

- **n8n Instance:** An active n8n environment.
- **GitHub API Credentials:** A GitHub credential configured in n8n with sufficient permissions to read and modify workflow settings (specifically the `workflow` scope).
- **Workflow Configuration:**
  - **Define Repositories:** Update the "Define Repositories" node with the array of repository names you wish to monitor.
  - **Dynamic Mapping (Required for Looping):** In the current version, the `repository` and `workflowId` parameters in the GitHub nodes (e.g., "Get Workflows for Repositories", "Enable Workflow", "Recheck Workflow") are hardcoded. To make the loop functional across all repositories, you must update these fields to use expressions (e.g., `{{ $json.repository }}`) instead of static values.

## How It Works

1. **Trigger:** The workflow can be triggered manually via the "Execute Workflow" button or automatically via a scheduled cron job (configured by default to run daily at midnight).
2. **Define Repositories:** A list of target repository names is provided in a `Set` node.
3. **Iteration:** The workflow splits the repository list into individual items and loops through each one.
4. **Status Check:** It fetches the details of a specific workflow for the current repository using the GitHub API.
5. **Condition Check:** It evaluates if the workflow's state is `active`.
6. **Automatic Remediation:** If the workflow is found to be in any state other than `active`, the workflow triggers an "Enable Workflow" action via the GitHub API.
7. **Verification:** After attempting to enable the workflow, it performs a follow-up check to confirm the status has been successfully updated.

## Expected Outcome

- **Automated Maintenance:** Reduces the manual effort required to monitor the health of CI/CD pipelines.
- **Self-Healing Pipelines:** If a workflow is accidentally disabled, this automation will detect the change and restore it during its next scheduled run, ensuring minimal disruption to development workflows.

## Workflow Design

The workflow is architected around a robust looping mechanism to handle multiple repositories efficiently:

- **Triggering Layer:** Employs both manual and scheduled triggers (`Daily @ Midnight`) to allow for both on-demand testing and automated periodic maintenance.
- **Data Initialization:** A `Set` node defines the scope of the monitor by providing an array of target repositories.
- **Batch Processing & Iteration:**
  - The array is flattened using `splitOut`, allowing each repository to be treated as an independent item.
  - A `splitInBatches` node (named "Loop Over Elements") manages the iteration, ensuring the workflow processes repositories one by one.
- **Conditional Logic & Remediation Loop:**
  - For each repository, the workflow queries the GitHub API.
  - An `IF` node acts as a decision gate, checking the `state` property of the retrieved workflow.
  - **Remediation Path:** If the state is not `active`, the workflow executes an "Enable" command and follows up with a "Recheck" node to ensure the action was successful before proceeding.
  - **Standard Path:** If the workflow is already active, it skips remediation and moves directly to the next item in the loop.
- **Loop Completion:** The connections ensure that after either a successful remediation or a check on an already-active workflow, control is returned to the batching node until all repositories have been processed.
