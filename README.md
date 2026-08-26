# Movie Picture Pipeline

## Project Overview

Movie Picture Pipeline is a DevOps CI/CD project that automates the testing, building, containerization, and deployment of a movie catalog web application.

The project contains two applications:

1. **Frontend** - React application
2. **Backend** - Python Flask REST API

GitHub Actions is used to implement Continuous Integration and Continuous Deployment pipelines.

Docker images are stored in Amazon Elastic Container Registry (ECR), and the applications are deployed to an Amazon Elastic Kubernetes Service (EKS) cluster using Kubernetes.

---

## Project Architecture

The overall deployment architecture is:

```text
Developer
    |
    v
GitHub Repository
    |
    v
Feature Branch
    |
    v
Pull Request
    |
    v
GitHub Actions CI
    |
    +---- Lint
    |
    +---- Test
    |
    v
Docker Build
    |
    v
Merge to main
    |
    v
GitHub Actions CD
    |
    +---- Lint
    |
    +---- Test
    |
    v
Docker Build
    |
    v
Amazon ECR
    |
    v
Amazon EKS
    |
    v
Kubernetes
    |
    +--------------------+
    |                    |
    v                    v
Frontend              Backend
LoadBalancer          LoadBalancer
```

---

# Technologies Used

The project uses the following technologies:

- Git
- GitHub
- GitHub Actions
- Docker
- React
- Node.js
- Python
- Flask
- Pipenv
- AWS
- Amazon ECR
- Amazon EKS
- Kubernetes
- kubectl
- Kustomize
- Terraform

---

# Project Structure

Important project directories and files:

```text
cd12354-Movie-Picture-Pipeline/
│
├── .github/
│   └── workflows/
│       ├── frontend-ci.yaml
│       ├── backend-ci.yaml
│       ├── frontend-cd.yaml
│       └── backend-cd.yaml
│
├── starter/
│   ├── frontend/
│   │   ├── k8s/
│   │   ├── src/
│   │   ├── Dockerfile
│   │   └── package.json
│   │
│   └── backend/
│       ├── k8s/
│       ├── movies/
│       ├── Dockerfile
│       ├── Pipfile
│       ├── Pipfile.lock
│       └── test_app.py
│
├── setup/
│   └── terraform/
│
├── screenshots/
│
└── README.md
```

---

# GitHub Actions Workflows

Four GitHub Actions workflows were implemented.

| Workflow | File | Automatic Trigger |
|---|---|---|
| Frontend Continuous Integration | `frontend-ci.yaml` | Pull request to `main` |
| Backend Continuous Integration | `backend-ci.yaml` | Pull request to `main` |
| Frontend Continuous Deployment | `frontend-cd.yaml` | Push/merge to `main` |
| Backend Continuous Deployment | `backend-cd.yaml` | Push/merge to `main` |

All four workflows also support manual execution using:

```yaml
workflow_dispatch:
```

---

# Frontend Continuous Integration

Workflow:

```text
.github/workflows/frontend-ci.yaml
```

The workflow is named:

```text
Frontend Continuous Integration
```

The Frontend CI workflow runs automatically whenever frontend changes are included in a pull request against the `main` branch.

The workflow also supports manual execution.

## Frontend CI Pipeline

```text
Pull Request to main
        |
   +----+----+
   |         |
   v         v
 Lint       Test
   |         |
   +----+----+
        |
        v
      Build
```

### Lint Job

The lint job performs:

1. Checkout source code
2. Setup Node.js
3. Restore/cache dependencies
4. Install dependencies
5. Run linting

Lint command:

```bash
npm run lint
```

### Test Job

The test job performs:

1. Checkout source code
2. Setup Node.js
3. Restore/cache dependencies
4. Install dependencies
5. Run tests

Tests are executed in the CI environment.

### Parallel Execution

The lint and test jobs execute in parallel.

### Build Job

The build job waits for both lint and test jobs to complete successfully.

The dependency is controlled using the GitHub Actions `needs` directive.

The application is then built using Docker.

If linting or testing fails, the build job does not proceed.

---

# Backend Continuous Integration

Workflow:

```text
.github/workflows/backend-ci.yaml
```

The workflow is named:

```text
Backend Continuous Integration
```

The Backend CI workflow runs automatically when backend changes are included in a pull request against the `main` branch.

Manual execution is also supported.

## Backend CI Pipeline

```text
Pull Request to main
        |
   +----+----+
   |         |
   v         v
 Lint       Test
   |         |
   +----+----+
        |
        v
      Build
```

### Backend Lint

The backend uses Flake8 through Pipenv.

```bash
pipenv run lint
```

### Backend Tests

The backend tests are executed using:

```bash
pipenv run test
```

The backend test suite verifies the Flask `/movies` API.

The application includes tests that verify:

- The endpoint returns HTTP 200
- The endpoint returns JSON
- The endpoint returns valid movie data

### Backend Build

The Docker build executes only after both linting and testing complete successfully.

---

# Frontend Continuous Deployment

Workflow:

```text
.github/workflows/frontend-cd.yaml
```

The workflow is named:

```text
Frontend Continuous Deployment
```

The Frontend CD workflow runs automatically when frontend changes are pushed or merged into the `main` branch.

Manual execution is also available.

## Frontend CD Pipeline

```text
Push / Merge to main
        |
   +----+----+
   |         |
   v         v
 Lint       Test
   |         |
   +----+----+
        |
        v
      Build
        |
        v
   Push to ECR
        |
        v
      Deploy
        |
        v
    Amazon EKS
```

### Frontend Docker Build

The frontend is built using Docker.

The backend API URL is passed into the Docker build using the build argument:

```text
REACT_APP_MOVIE_API_URL
```

This allows the deployed frontend application to communicate with the backend API running through the backend Kubernetes LoadBalancer.

### Docker Image Tagging

Docker images are tagged using the Git commit SHA.

This allows each Docker image to be traced back to the commit that generated it.

### Amazon ECR

The Docker image is pushed to the frontend Amazon ECR repository.

The workflow uses the AWS ECR login action to authenticate with Amazon ECR.

### Kubernetes Deployment

After the image is successfully pushed to ECR, the deployment job updates the Kubernetes manifest with the newly generated image.

Kustomize and kubectl are used to deploy the application to Amazon EKS.

---

# Backend Continuous Deployment

Workflow:

```text
.github/workflows/backend-cd.yaml
```

The workflow is named:

```text
Backend Continuous Deployment
```

The Backend CD workflow runs automatically when backend changes are pushed or merged into the `main` branch.

Manual execution is also supported.

## Backend CD Pipeline

```text
Push / Merge to main
        |
   +----+----+
   |         |
   v         v
 Lint       Test
   |         |
   +----+----+
        |
        v
      Build
        |
        v
   Push to ECR
        |
        v
      Deploy
        |
        v
    Amazon EKS
```

The workflow:

1. Runs backend linting
2. Runs backend tests
3. Builds the backend Docker image
4. Authenticates with AWS
5. Logs into Amazon ECR
6. Tags the image using the Git SHA
7. Pushes the Docker image to ECR
8. Connects to Amazon EKS
9. Updates the Kubernetes image
10. Deploys the application using Kubernetes

---

# Automatic CI/CD Trigger Verification

Automatic triggering of all pipelines was verified using separate Git branches and pull requests.

## Frontend CI Verification

A branch named:

```text
frontend-test
```

was created.

A frontend change was committed and pushed to the branch.

A pull request was then opened:

```text
frontend-test -> main
```

GitHub automatically started:

```text
Frontend Continuous Integration
```

The following checks passed:

```text
Lint  -> Passed
Test  -> Passed
Build -> Passed
```

The pull request was successfully merged after all checks passed.

---

# Frontend CD Verification

Merging the frontend pull request caused a push to `main`.

GitHub automatically started:

```text
Frontend Continuous Deployment
```

The workflow showed:

```text
Triggered via push

Lint   -> Passed
Test   -> Passed
Build  -> Passed
Deploy -> Passed
```

This verified that the frontend CD pipeline runs automatically after frontend changes are merged into `main`.

---

# Backend CI Verification

A separate branch named:

```text
backend-test
```

was created.

A backend change was committed and pushed to the branch.

A pull request was opened:

```text
backend-test -> main
```

GitHub automatically started:

```text
Backend Continuous Integration
```

The following checks completed successfully:

```text
Lint  -> Passed
Test  -> Passed
Build -> Passed
```

The pull request showed:

```text
All checks have passed
3 successful checks
No conflicts with base branch
```

The pull request was then merged into `main`.

---

# Backend CD Verification

After merging the backend pull request, GitHub automatically started:

```text
Backend Continuous Deployment
```

The workflow was triggered by the push to `main`.

All deployment stages passed:

```text
Lint   -> Passed
Test   -> Passed
Build  -> Passed
Deploy -> Passed
```

This verified the complete automatic Backend CI/CD process.

---

# AWS Infrastructure

AWS infrastructure for the deployment was created using Terraform.

Terraform created/configured the resources required for:

- Amazon EKS
- Amazon ECR
- IAM access for GitHub Actions
- Kubernetes deployment

## Terraform Outputs

The environment produced the following important outputs:

```text
cluster_name = "cluster"
cluster_version = "1.34"

frontend_ecr =
017747176708.dkr.ecr.us-east-1.amazonaws.com/frontend

backend_ecr =
017747176708.dkr.ecr.us-east-1.amazonaws.com/backend

github_action_user_arn =
arn:aws:iam::017747176708:user/github-action-user
```

AWS region:

```text
us-east-1
```

---

# Amazon ECR

Two private ECR repositories are used.

## Frontend Repository

```text
frontend
```

The frontend CD workflow successfully pushes the React Docker image to this repository.

## Backend Repository

```text
backend
```

The backend CD workflow successfully pushes the Flask Docker image to this repository.

Docker images are tagged using their corresponding Git commit SHA.

---

# GitHub Secrets

AWS credentials required by GitHub Actions are stored securely using GitHub repository secrets.

Credentials are not hard-coded directly inside the workflow files.

GitHub Actions retrieves the credentials from GitHub Secrets when authenticating with AWS.

This allows the workflows to securely:

- Authenticate with AWS
- Login to Amazon ECR
- Push Docker images
- Access Amazon EKS
- Run Kubernetes deployments

---

# Amazon EKS

The applications are deployed to an Amazon EKS cluster.

Cluster name:

```text
cluster
```

The Kubernetes cluster contains both frontend and backend deployments.

The cluster status can be verified using:

```bash
kubectl get nodes
```

---

# Kubernetes

Kubernetes is used to run both applications.

## Pods

Deployment status can be checked using:

```bash
kubectl get pods
```

Both application pods reached the `Running` state.

Example:

```text
NAME                         READY   STATUS    RESTARTS
backend-xxxxxxxxxx-xxxxx     1/1     Running   0
frontend-xxxxxxxxxx-xxxxx    1/1     Running   0
```

---

## Deployments

Deployments can be checked using:

```bash
kubectl get deployments
```

Both applications have an available replica:

```text
NAME       READY   UP-TO-DATE   AVAILABLE
backend    1/1     1            1
frontend   1/1     1            1
```

---

## Services

Services can be checked using:

```bash
kubectl get services
```

The project contains:

```text
backend      LoadBalancer
frontend     LoadBalancer
kubernetes   ClusterIP
```

Both frontend and backend are exposed through AWS LoadBalancers.

---

# Application Verification

## Backend API

The backend Flask API exposes the movies endpoint:

```text
/movies
```

The deployed API successfully returned:

```json
{
  "movies": [
    {
      "id": "123",
      "title": "Top Gun: Maverick"
    },
    {
      "id": "456",
      "title": "Sonic the Hedgehog"
    },
    {
      "id": "789",
      "title": "A Quiet Place"
    }
  ]
}
```

This verifies that the backend application is successfully running in Kubernetes.

---

# Frontend Application

The React frontend was successfully deployed through the Kubernetes frontend LoadBalancer.

The application displays:

```text
Movie List

• Top Gun: Maverick
• Sonic the Hedgehog
• A Quiet Place
```

The movie information is retrieved from the deployed backend API.

Therefore, the frontend deployment also verifies that:

```text
REACT_APP_MOVIE_API_URL
```

was correctly provided during the frontend Docker build.

---

# Screenshots

Project verification screenshots are stored in:

```text
screenshots/
```

The screenshots provide evidence for:

1. Frontend Continuous Integration
2. Backend Continuous Integration
3. Frontend Continuous Deployment
4. Backend Continuous Deployment
5. Automatic CI pull-request execution
6. Automatic CD push execution
7. Frontend application running
8. Backend `/movies` API response
9. Frontend ECR image
10. Backend ECR image
11. Kubernetes pods
12. Kubernetes deployments
13. Kubernetes services

---

# Frontend Development

## Install Dependencies

Navigate to the frontend directory:

```bash
cd starter/frontend
```

Install dependencies:

```bash
npm ci
```

---

## Run Frontend Tests

Run:

```bash
CI=true npm test
```

Expected successful tests include:

```text
PASS src/components/__tests__/MovieList.test.js
PASS src/components/__tests__/App.test.js

Test Suites: 2 passed
Tests:       3 passed
```

---

## Run Frontend Linter

```bash
npm run lint
```

The linting step must complete without ESLint errors.

---

## Run Frontend Locally

```bash
REACT_APP_MOVIE_API_URL=http://localhost:5000 npm start
```

---

# Backend Development

Navigate to:

```bash
cd starter/backend
```

## Install Dependencies

```bash
pipenv install
```

---

## Run Backend Tests

```bash
pipenv run test
```

The backend tests verify:

```text
test_movies_endpoint_returns_200
test_movies_endpoint_returns_json
test_movies_endpoint_returns_valid_data
```

All tests must pass before the Docker build/deployment can continue.

---

## Run Backend Linter

```bash
pipenv run lint
```

---

## Run Backend Locally

```bash
pipenv run serve
```

The application runs on port:

```text
5000
```

The API can then be tested using:

```bash
curl http://localhost:5000/movies
```

---

# Docker

## Build Frontend Image

From the frontend directory:

```bash
docker build \
  --build-arg REACT_APP_MOVIE_API_URL=http://localhost:5000 \
  --tag mp-frontend:latest .
```

---

## Build Backend Image

From the backend directory:

```bash
docker build --tag mp-backend:latest .
```

---

# Kubernetes Deployment with Kustomize

## Frontend

Navigate to:

```bash
cd starter/frontend/k8s
```

Set the image:

```bash
kustomize edit set image frontend=<ECR_REPO_URL>:<GIT_SHA>
```

Deploy:

```bash
kustomize build | kubectl apply -f -
```

---

## Backend

Navigate to:

```bash
cd starter/backend/k8s
```

Set the image:

```bash
kustomize edit set image backend=<ECR_REPO_URL>:<GIT_SHA>
```

Deploy:

```bash
kustomize build | kubectl apply -f -
```

---

# CI/CD Failure Protection

The pipelines are designed so that deployment cannot continue when required validation fails.

The execution dependency is:

```text
Lint -----+
          |
          +----> Build ----> Deploy
          |
Test -----+
```

Therefore:

- If lint fails -> build does not continue.
- If tests fail -> build does not continue.
- If build fails -> deployment does not continue.
- Deployment occurs only after the required previous stages succeed.

This prevents broken application changes from being automatically deployed.

---

# Security

The project follows these credential-management practices:

- AWS credentials are stored in GitHub Secrets.
- AWS credentials are not hard-coded in workflow YAML files.
- GitHub Actions accesses AWS credentials only when required.
- Docker images are stored in private Amazon ECR repositories.
- Kubernetes access is controlled through AWS/EKS authentication.

---

# Final Project Results

| Requirement | Status |
|---|---|
| Frontend CI workflow | ✅ Completed |
| Backend CI workflow | ✅ Completed |
| Frontend CD workflow | ✅ Completed |
| Backend CD workflow | ✅ Completed |
| Automatic Frontend CI trigger | ✅ Verified |
| Automatic Backend CI trigger | ✅ Verified |
| Automatic Frontend CD trigger | ✅ Verified |
| Automatic Backend CD trigger | ✅ Verified |
| Frontend linting | ✅ Passed |
| Frontend tests | ✅ Passed |
| Backend linting | ✅ Passed |
| Backend tests | ✅ Passed |
| Frontend Docker build | ✅ Passed |
| Backend Docker build | ✅ Passed |
| Frontend image pushed to ECR | ✅ Verified |
| Backend image pushed to ECR | ✅ Verified |
| Frontend deployed to EKS | ✅ Running |
| Backend deployed to EKS | ✅ Running |
| Kubernetes pods | ✅ Running |
| Kubernetes deployments | ✅ Available |
| Kubernetes LoadBalancers | ✅ Available |
| Frontend movie list | ✅ Working |
| Backend movies API | ✅ Working |

---

# Conclusion

The Movie Picture Pipeline project successfully implements an automated CI/CD process for both the frontend and backend applications.

Pull requests automatically trigger Continuous Integration pipelines that perform linting, testing, and Docker builds.

After approved changes are merged into the `main` branch, Continuous Deployment pipelines automatically build Docker images, push them to Amazon ECR, and deploy the latest application versions to Amazon EKS.

The final frontend and backend applications were successfully deployed to Kubernetes, and communication between the React frontend and Flask backend was verified.

All four GitHub Actions workflows completed successfully.