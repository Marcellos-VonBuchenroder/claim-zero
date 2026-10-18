# Claim Zero: React App to AWS with Docker, Terraform and CI/CD

A React (Vite) app that I containerised, published to Amazon ECR, deployed to Amazon ECS with Terraform, and connected to an AWS CodePipeline that rebuilds and redeploys on every push to `main`. The AWS resources were torn down after the course to stay within the Free Tier.

This was built during the **Codewrkx Introduction to Cloud Engineering (AWS)** accelerator (September 2026). It was a guided, hands-on course: the app and lab steps were provided, and I did the packaging, provisioning and pipeline work myself on my own AWS account.

## Architecture

```text
 GitHub push
     │
     ▼
 AWS CodePipeline ──► CodeBuild (buildspec.yml)
                         │  docker build → tag → push
                         ▼
                    Amazon ECR  (claim-zero:latest)
                         │  imagedefinitions.json
                         ▼
                    Amazon ECS (EC2 launch type, t3.micro)
                         │  nginx serving the Vite build on port 80
                         ▼
                    Public URL ── logs → CloudWatch
```

## What's in this repo

| Path | Purpose |
|---|---|
| `Dockerfile` | Multi-stage build: Node 22 builds the Vite app, then nginx (alpine) serves the static output, which keeps the final image small |
| `.dockerignore` | Keeps `node_modules`, Git data and local files out of the build context |
| `terraform/main.tf` | Infrastructure as code: security group, ECS cluster, EC2 container host, IAM roles (instance + task execution), CloudWatch log group, task definition and ECS service |
| `buildspec.yml` | CodeBuild steps: log in to ECR, build and tag the image, push it, and write `imagedefinitions.json` for the ECS deploy stage |
| `docs/` | The course lab notes for days 1 to 3 (package, publish and provision, automate). The pipeline Terraform (`pipeline.tf`) is in `docs/day-3.md` |

## Skills practised

- **Docker:** multi-stage builds, image tagging, running and cleaning up containers
- **AWS:** IAM users and roles, ECR, ECS on EC2, CloudWatch Logs, CodeBuild and CodePipeline
- **Terraform:** provider pinning, data sources (default VPC, SSM-published ECS AMI), resource dependencies and outputs
- **CI/CD:** a push-to-deploy pipeline with no manual steps after setup

## Run it locally

```bash
docker build -t claim-zero:latest .
docker run --rm -p 8080:80 claim-zero:latest
# open http://localhost:8080
```

## Deploy (summary)

```bash
# 1. Push the image to ECR (see docs/day-2.md)
# 2. Provision the infrastructure
cd terraform
terraform init
terraform apply      # prints app_url when done
# 3. Tear down when finished, to stay within the Free Tier
terraform destroy
```

> **Note:** this is a learning setup. The security group allows HTTP from anywhere, and there is no load balancer or HTTPS. A production version would add an ALB with TLS, private subnets and tighter ingress rules.
