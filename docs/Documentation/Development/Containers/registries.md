# Container registries at NLR

## Introduction
Container registries enable users to store container images. An overview of the steps to use each fo the main container registries available to NLR users is provided below. Registries can enable reproducibility by storing tagged versions of containers, and also facilitate transferring images easily between different computational resources. 

## Create Docker images
Docker is not supported on NLR's HPC systems, including Kestrel. Instead, [Apptainer](apptainer.md) is the container engine provided as a module. Apptainer is able to pull Docker images and convert them to Apptainer-formatted images. We generally recommend [building Docker images](index.md#example-docker-build-workflow-for-hpc-users) to ensure portability between compute resources and using Apptainer to convert the image when running on an HPC system. 

## Accessibility

| Registry  | Kestrel Access | AWS Access | Docker Support | Apptainer Support   | Podman Support |
| --------- | -------------- | ---------- | -------------- | ------------------- | -------------- |
| Pulp      | No**           | No         | Yes            | Yes                 | Yes            | 
| AWS ECR   | Yes            | Yes        | Yes            | No*                 | Yes            |
| DockerHub | Yes            | Yes        | Yes            | No*                 | Yes            |
*for DockerHub and AWS ECR it may be possible to push images using ORAS, but this was not found to be a streamlined process in testing. 
**Pulp is under active development. Please check back regularly on this page for updates.

## Pulp
NLR's internal "Pulp" registry supports Docker, Apptainer, and Podman containers. It is under active development and is currently only available to a small number of users. If you would like to work with Pulp, please reach out to [hpc-help@nlr.gov](hpc-help@nlr.gov).

## AWS ECR
AWS ECR can be utilized by projects with a cloud allocation to host containers. ECR primarily can be used with Docker containers, although Apptainer should also be possible. 

## DockerHub
[DockerHub](https://hub.docker.com/) is a free, public container registry. NLR HPC users are encouraged to refer to the [official Docker documentation](https://docs.docker.com/docker-hub/quickstart/) on how to effectively use DockerHub as a registry.