# Prisma Cloud Demo

The repository has 3 main folders: Terraform, DockerImages and Kubernetes and contains IaC necessary to:

- Provision an EKS Cluster and Bastion EC2 instance via Terraform.
- Build Docker images necessary to run the sample applications.
- Deploy a few sample applications on a Kubernetes cluster.

**Note**: The code isn't production ready and is intentionally misconfigured. This is only for demonstration purposes.

## Purpose

The Github Actions workflow in this repo is designed to perform Prisma Cloud Scans on:
- Code to build Docker images
- Terraform code
- Kubernetes manifests

1. The idea is to bring all the components of Prisma Cloud Scan together (Twistcli image scan, Twistcli sandbox scan and Bridgecrew scan) to scan differnt pieces of IaC in the same CICD run.
2. This also demonstrates how we can integrate Github Actions and Prisma Cloud 
3. Development branch would typically contain code that's misconfigured and contains vulnerabilities. Scans are performed against this branch to reveal these misconfigurations.
4. QA branch in this repo would contain code this is fixed, meaning, all the misconfigurations that were highlighted by Prisma Cloud in the Development branch
