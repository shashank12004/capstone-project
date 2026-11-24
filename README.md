# QuickBite -- End-to-End DevOps Automation

**Course:** DevOps & Automation Lab (ENSP461)\
**Semester:** VII\
**Project:** QuickBite -- Online Food Ordering Platform\
**Prepared by:** \[Your Name\]\
**Roll Number:** \[Your Roll Number\]\
**GitHub Repository:** \[Add repository URL\]

> **Status note:** This README covers all 11 project phases. It is a
> complete implementation guide and deliverables checklist, not a claim
> that every phase is already implemented. Update the status table and
> add genuine screenshots/results before submission.

## 1. Project Overview

QuickBite is an online food-ordering application used to demonstrate an
end-to-end DevOps lifecycle. The application has a **Next.js/React
frontend**, a **Java Spring Boot REST backend**, and a **MySQL
database**. The DevOps workflow covers source control, CI, containers,
Kubernetes, configuration management, Infrastructure as Code,
monitoring, logging, security, release strategies and GitOps.

### Objectives

1.  Track application and deployment changes in Git.
2.  Automate build and validation using Jenkins.
3.  Package frontend and backend using Docker.
4.  Deploy and manage workloads using Kubernetes.
5.  Automate configuration with Ansible.
6.  Provision cloud infrastructure with Terraform.
7.  Monitor application and cluster health.
8.  Centralize logs for troubleshooting.
9.  Apply DevSecOps and access-control practices.
10. Demonstrate controlled deployments and rollback.
11. Manage desired cluster state through GitOps.

## 2. Architecture

``` text
Developer -> GitHub -> Jenkins CI -> Docker build -> Image Registry
                                            |
                                            v
                                    Kubernetes Cluster
                                     /      |       \
                                Frontend  Backend  MySQL
                                             |
                                      Persistent Storage

Ansible: host/configuration automation
Terraform: infrastructure provisioning
Prometheus + Grafana: metrics and dashboards
Fluentd / ELK: centralized logs
SonarQube / Trivy: quality and image scanning
Argo CD / Flux: GitOps reconciliation
```

### Environments

  -----------------------------------------------------------------------
  Environment             Purpose                 Components
  ----------------------- ----------------------- -----------------------
  Local development       Build and test          Docker Compose,
                                                  frontend, backend,
                                                  MySQL

  Local Kubernetes        Instructor              Docker Desktop
                          demonstration           Kubernetes,
                                                  Deployments, Services,
                                                  PVC

  AWS learning path       Cloud extension         EKS, ECR, RDS, VPC,
                                                  security groups,
                                                  CloudWatch
  -----------------------------------------------------------------------

The local Docker Desktop Kubernetes environment is the primary demo
target. Treat AWS EKS as a learning extension until it is actually
deployed and verified.

## 3. Technology Stack

  Area                       Technology
  -------------------------- --------------------------------------------------------
  Frontend                   Next.js, React
  Backend                    Java, Spring Boot, REST
  Database                   MySQL
  Source control             Git, GitHub
  CI                         Jenkins
  Build                      Maven, npm
  Containers                 Docker, Docker Compose
  Orchestration              Kubernetes, kubectl
  Configuration management   Ansible
  Infrastructure as Code     Terraform
  Monitoring                 Prometheus, Grafana
  Logging                    Fluentd and/or ELK
  Security                   SonarQube, Trivy, Jenkins Credentials, Kubernetes RBAC
  GitOps                     Argo CD or Flux CD
  Cloud extension            AWS EKS, ECR, RDS, VPC

Only list tools in the final report that you actually configured or
demonstrated. Label other tools as planned.

------------------------------------------------------------------------

# 4. Phase-wise Implementation

## Phase 1 -- Version Control with Git and GitHub

**Objective:** Maintain source history, collaboration and a shared
source of truth.

**Implementation:** 1. Create a GitHub repository for QuickBite. 2.
Initialize Git in the project root. 3. Add frontend, backend,
Dockerfiles, Jenkinsfile, Kubernetes manifests and documentation. 4. Add
`.gitignore` for generated files, build output, local environment files
and secrets. 5. Commit incremental changes with meaningful messages. 6.
Push to GitHub and use feature branches for new work.

**PowerShell commands:**

``` powershell
cd path\to\quickbite
git init
git branch -M main
git add .
git commit -m "Initial QuickBite project"
git remote add origin https://github.com/<username>/<repository>.git
git push -u origin main
```

If a remote already exists:

``` powershell
git remote -v
git remote set-url origin https://github.com/<username>/<repository>.git
git push -u origin main
```

**Deliverables:** GitHub repository URL, commit history, branch/merge
evidence, `.gitignore`, project README.

**Evidence:** Repository page, commits, branches or pull request,
successful push output.

## Phase 2 -- Continuous Integration with Jenkins

**Objective:** Automatically build and validate application changes.

**Implementation:** 1. Install and start Jenkins. 2. Ensure the Jenkins
agent has compatible Git, JDK, Maven and Node.js/npm installations. 3.
Add a `Jenkinsfile` at the repository root. 4. Create a Pipeline job
using **Pipeline script from SCM**. 5. Configure repository URL, branch
(for example `*/main`) and script path (`Jenkinsfile`). 6. Configure a
GitHub webhook or SCM polling. 7. Run the pipeline and review stages,
console output and test results.

**Typical pipeline:**

``` text
Checkout -> Backend Maven tests/package -> Frontend npm install/build
         -> Optional quality/security checks -> Archive artifacts
```

**Example validation commands:**

``` powershell
# Backend
cd backend
mvn -B clean test package

# Frontend
cd ..\frontend
npm install
npm run build
```

Use working directories and commands that match your actual repository.

**Deliverables:** `Jenkinsfile`, configured pipeline, SCM trigger,
successful build/test output, archived artifacts if configured.

**Evidence:** Jenkins job configuration, pipeline stages, console
output, test results and archived artifact.

## Phase 3 -- Containerization with Docker

**Objective:** Package frontend and backend into repeatable images.

**Implementation:** 1. Create a backend Dockerfile for the Spring Boot
application. 2. Create a frontend Dockerfile for Next.js. 3. Build and
tag images. 4. Use Docker Compose to run frontend, backend and MySQL
locally. 5. Configure ports and environment variables. 6. Verify
container status, logs and network connectivity. 7. Do not store
production secrets in Dockerfiles or source control.

**Example commands:**

``` powershell
docker build -t quickbite-backend:local ./backend
docker build --build-arg NEXT_PUBLIC_API_URL=http://localhost:8081 -t quickbite-frontend:local ./frontend
docker image ls
docker compose up --build -d
docker compose ps
docker compose logs --tail=100
```

Confirm how the frontend constructs API paths before deciding whether
the API base URL should include `/api`. `NEXT_PUBLIC_` values are
generally embedded at frontend build time, so rebuild after changing
them.

**Deliverables:** Frontend and backend Dockerfiles, Compose file, built
image tags, running containers.

**Evidence:** Docker Desktop container list, image list,
`docker compose ps`, application in browser.

## Phase 4 -- Container Orchestration with Kubernetes

**Objective:** Deploy, scale and recover application workloads
declaratively.

**Local target:** Docker Desktop Kubernetes.

**Resources:**

  -----------------------------------------------------------------------
  Resource                            Purpose
  ----------------------------------- -----------------------------------
  Namespace                           Groups QuickBite resources

  Deployment                          Manages Pod replicas

  Service                             Stable networking to Pods

  ConfigMap                           Non-sensitive configuration

  Secret                              Sensitive runtime configuration

  PersistentVolumeClaim               Persistent database storage

  Startup/readiness/liveness probes   Control startup, traffic
                                      eligibility and recovery
  -----------------------------------------------------------------------

**Suggested structure:**

``` text
k8s/
  namespace.yaml
  mysql/
    mysql-secret.yaml
    mysql-config.yaml
    mysql-pvc.yaml
    mysql-deployment.yaml
    mysql-service.yaml
  backend/
    backend-config.yaml
    backend-deployment.yaml
    backend-service.yaml
  frontend/
    frontend-deployment.yaml
    frontend-service.yaml
```

**Apply manifests:**

``` powershell
kubectl config current-context
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/mysql/
kubectl apply -f k8s/backend/
kubectl apply -f k8s/frontend/
kubectl get pods -n quickbite -o wide
kubectl get services -n quickbite
kubectl get deployments -n quickbite
```

**Port forwarding:** Run each command in a separate PowerShell window
and keep it open. Confirm actual Service names and ports in your
manifests.

``` powershell
kubectl port-forward service/quickbite-frontend 3000:3000 -n quickbite
kubectl port-forward service/quickbite-backend 8081:8080 -n quickbite
```

**Scaling and rollout:**

``` powershell
kubectl scale deployment/quickbite-backend --replicas=3 -n quickbite
kubectl rollout status deployment/quickbite-backend -n quickbite
kubectl rollout history deployment/quickbite-backend -n quickbite
kubectl rollout undo deployment/quickbite-backend -n quickbite
```

**Troubleshooting:**

``` powershell
kubectl describe pod <pod-name> -n quickbite
kubectl logs <pod-name> -n quickbite
kubectl logs <pod-name> -n quickbite --previous
kubectl get events -n quickbite --sort-by=.metadata.creationTimestamp
kubectl get endpoints -n quickbite
```

Slow-starting Spring Boot or MySQL containers may need startup probes
and realistic timeouts. Diagnose Pod events and previous logs before
changing application code. Do not delete the database PVC as a routine
troubleshooting step.

**Deliverables:** Namespace, Deployments, Services, ConfigMaps, Secrets,
PVC, probes, scaling/rollout evidence.

**Evidence:** Pod and Service output, application accessed through
port-forward, scaling and rollout results.

## Phase 5 -- Configuration Management with Ansible

**Objective:** Automate repeatable host setup and configuration.

**Implementation:** 1. Create an inventory of target hosts. 2. Write
playbooks for prerequisite installation and configuration. 3. Use
variables and templates for environment-specific settings. 4. Use
handlers to restart services only when configuration changes. 5.
Organize reusable tasks into roles where useful. 6. Run the playbook and
verify the target state. 7. Run it again to demonstrate idempotency.

**Suggested structure:**

``` text
ansible/
  ansible.cfg
  inventory/hosts.ini
  playbooks/setup.yml
  roles/
    common/
      tasks/main.yml
      handlers/main.yml
      templates/
```

**Example commands:**

``` bash
ansible --version
ansible-inventory -i ansible/inventory/hosts.ini --list
ansible-playbook -i ansible/inventory/hosts.ini ansible/playbooks/setup.yml --check
ansible-playbook -i ansible/inventory/hosts.ini ansible/playbooks/setup.yml
```

These commands require Ansible and reachable target hosts. Windows users
commonly run Ansible from WSL or a Linux control node.

**Deliverables:** Inventory, playbooks, roles, variables/templates,
successful execution and idempotency evidence.

**Evidence:** Inventory and playbook files, successful run output,
target verification.

## Phase 6 -- Infrastructure as Code with Terraform

**Objective:** Define infrastructure in version-controlled
configuration.

**AWS learning architecture:** VPC, public/private subnets, routes,
security groups, EKS cluster and node groups, ECR repositories, RDS
MySQL, IAM roles and CloudWatch integration.

**Suggested structure:**

``` text
terraform/
  versions.tf
  providers.tf
  variables.tf
  main.tf
  outputs.tf
  terraform.tfvars.example
  modules/
    network/
    eks/
    ecr/
    database/
```

**Example commands:**

``` bash
terraform init
terraform fmt -check
terraform validate
terraform plan
```

Review the plan, account, region, permissions and estimated costs before
provisioning:

``` bash
terraform apply
```

Destroy only resources you intentionally created and no longer need:

``` bash
terraform destroy
```

**Good practices:** - Never commit AWS access keys, passwords or
secret-bearing `tfvars`. - Exclude state files and local secret files
from Git. - Use remote state and locking for team/production use. -
Review EKS and RDS costs before provisioning. - Use Terraform outputs
instead of guessing resource identifiers.

**Deliverables:** Provider/resource definitions, variables, outputs,
modules if used, validate/plan output, infrastructure diagram and state
approach.

**Evidence:** Terraform configuration, plan output, AWS console
resources if actually provisioned. Mask account IDs and secrets.

## Phase 7 -- Monitoring and Metrics

**Objective:** Observe application health, Kubernetes resources and
service performance.

**Implementation:** 1. Configure Prometheus to scrape supported metrics
targets. 2. Expose application metrics where configured. 3. Connect
Grafana to Prometheus. 4. Create dashboards for CPU, memory, Pod status,
restarts, request rate and latency where metrics exist. 5. Define alerts
for unavailable workloads or resource saturation. 6. Confirm dashboards
display live data from the intended targets.

**Suggested dashboard panels:** Pod availability, CPU/memory, restart
count, HTTP request rate/latency (if instrumented), backend health and
database health (if exposed).

**Deliverables:** Prometheus scrape configuration, Grafana dashboard,
healthy targets, metrics panels and alert rules.

**Evidence:** Prometheus target status, Grafana dashboard with real
data, alert evidence. Do not report fabricated metrics.

## Phase 8 -- Centralized Logging

**Objective:** Collect and search application and infrastructure logs
centrally.

**Implementation:** 1. Identify frontend, backend, database and
Kubernetes log sources. 2. Configure Fluentd or an ELK-based collection
pipeline. 3. Forward logs to a central store. 4. Parse timestamp,
service, severity and message fields. 5. Create searches or dashboards
for errors and operational events. 6. Demonstrate investigation of a
real request or deployment issue.

``` text
Application/container logs -> Collector -> Central log store -> Search/dashboard
```

**Deliverables:** Collector configuration, forwarding pipeline,
searchable log view, example investigation, retention/access approach.

**Evidence:** Collector configuration and central log search showing
service and timestamp.

## Phase 9 -- Security and DevSecOps

**Objective:** Reduce risks in source code, dependencies, images,
credentials and cluster access.

**Implementation:** 1. Add static analysis using SonarQube or an
equivalent tool. 2. Scan images with Trivy or an approved scanner. 3.
Scan application dependencies where supported. 4. Store CI credentials
in Jenkins Credentials. 5. Use Kubernetes Secrets for sensitive runtime
values. 6. Apply least-privilege Kubernetes RBAC. 7. Avoid unnecessary
service exposure. 8. Review base images and update vulnerable
dependencies.

**Security checklist:** - \[ \] No credentials committed to Git. - \[ \]
`.env` and local secret files excluded from source control. - \[ \]
Jenkins credentials stored in the credential store. - \[ \] Kubernetes
Secrets used for sensitive values. - \[ \] RBAC grants only required
permissions. - \[ \] Image scan run and reviewed. - \[ \]
Static-analysis report captured. - \[ \] Sensitive values masked in
screenshots and logs.

**Deliverables:** Code-quality report, image scan, secret-management
configuration, RBAC manifests and secure pipeline evidence.

**Evidence:** Analysis report, scanner output, RBAC configuration and
sanitized Secret manifest.

> Kubernetes Secrets are base64-encoded by default; this alone is not
> encryption. Protect cluster access and configure encryption at rest
> for production.

## Phase 10 -- Deployment Strategy: Blue-Green or Canary

**Objective:** Release changes with controlled exposure and a clear
rollback path.

**Blue-Green:** 1. Keep the current version (Blue) active. 2. Deploy a
candidate version (Green). 3. Validate Green with health and smoke
tests. 4. Switch Service/Ingress traffic to Green. 5. Keep Blue
available temporarily for rollback.

**Canary:** 1. Deploy the new version beside the stable version. 2.
Route a small percentage of traffic to it. 3. Monitor errors, latency
and health. 4. Gradually increase traffic if healthy. 5. Roll back if
defined thresholds are exceeded.

``` text
Build new version -> Deploy candidate -> Health/smoke tests
                              |
                      Traffic switch / ramp
                              |
                         Monitor
                        /       \
                    Healthy   Unhealthy
                      |          |
                   Promote    Roll back
```

**Deliverables:** Selected strategy and rationale, versioned deployment
configuration, traffic/rollout evidence, health validation and rollback
procedure.

**Evidence:** Before/after versions, routing configuration, rollout and
rollback output. Demonstrate one strategy unless both are required.

## Phase 11 -- GitOps with Argo CD or Flux CD

**Objective:** Store desired Kubernetes state in Git and continuously
reconcile the cluster.

**Implementation:** 1. Store manifests or Helm/Kustomize configuration
in Git. 2. Install Argo CD or Flux in the cluster. 3. Configure an
application/source pointing to the repository and deployment path. 4.
Connect it to the target namespace and cluster. 5. Synchronize desired
state. 6. Change a manifest in Git and observe reconciliation. 7.
Demonstrate drift detection and recovery. 8. Use a Git revert to roll
back a configuration change.

**Deliverables:** GitOps repository/folder, Argo CD Application or Flux
configuration, sync status, commit traceability, drift/reconciliation
and rollback evidence.

**Evidence:** Argo CD/Flux application view, Git commit, synced revision
and healthy resources. If not installed, label this phase as planned.

------------------------------------------------------------------------

# 5. Testing and Validation

Fill in actual results; do not assume a test passed.

  -----------------------------------------------------------------------
  Test area               Validation              Result
  ----------------------- ----------------------- -----------------------
  Frontend                Menu page loads         \[Pass/Fail/Not run\]

  Backend                 Menu API responds       \[Pass/Fail/Not run\]

  Orders                  Order endpoint and      \[Pass/Fail/Not run\]
                          persistence             

  Database                Backend connects to     \[Pass/Fail/Not run\]
                          MySQL                   

  CI                      Jenkins pipeline        \[Pass/Fail/Not run\]
                          completes               

  Docker                  Images build and        \[Pass/Fail/Not run\]
                          containers run          

  Kubernetes              Required Pods become    \[Pass/Fail/Not run\]
                          Ready                   

  Kubernetes              Services route to       \[Pass/Fail/Not run\]
                          healthy Pods            

  Storage                 Data survives Pod       \[Pass/Fail/Not run\]
                          replacement             

  Ansible                 Playbook runs           \[Pass/Fail/Not run\]
                          repeatably              

  Terraform               Configuration validates \[Pass/Fail/Not run\]
                          and plan reviewed       

  Monitoring              Live metrics appear     \[Pass/Fail/Not run\]

  Logging                 Logs searchable         \[Pass/Fail/Not run\]
                          centrally               

  Security                Code/image scans        \[Pass/Fail/Not run\]
                          reviewed                

  Release                 Strategy and rollback   \[Pass/Fail/Not run\]
                          demonstrated            

  GitOps                  Repository state        \[Pass/Fail/Not run\]
                          reconciled              
  -----------------------------------------------------------------------

Record test date, environment, application version and screenshot/log
reference for every test.

## 6. Troubleshooting Notes

### Port already in use

``` powershell
docker ps
netstat -ano | findstr :8080
```

Identify the process/container using the port. Choose a different host
port while keeping the container port unchanged.

### Docker Desktop engine unavailable

-   Confirm Docker Desktop is running and the Linux container engine has
    started.
-   Retry `docker version` and `docker info`.
-   Review Docker Desktop diagnostics if the daemon is unavailable.

### Frontend cannot reach backend

-   Confirm the backend container/Pod is running.
-   Test the backend endpoint directly.
-   Confirm the frontend API base URL and endpoint path match.
-   Rebuild the frontend after changing build-time `NEXT_PUBLIC_`
    variables.

### Kubernetes Pod is not Ready

``` powershell
kubectl get pods -n quickbite
kubectl describe pod <pod-name> -n quickbite
kubectl logs <pod-name> -n quickbite
kubectl logs <pod-name> -n quickbite --previous
kubectl get events -n quickbite --sort-by=.metadata.creationTimestamp
```

Check image availability, environment variables, DB connectivity, ports
and probes. Use startup probes for slow-starting applications. Do not
delete the database PVC unless you intentionally want to delete its
data.

## 7. Deliverables Summary

  -----------------------------------------------------------------------
  Phase                   Main deliverable        Evidence
  ----------------------- ----------------------- -----------------------
  1 Git/GitHub            Repository, commits,    Repository and commit
                          branches                screenshots

  2 Jenkins               Jenkinsfile and working Pipeline and console
                          pipeline                output

  3 Docker                Dockerfiles, Compose,   Images and running
                          images                  containers

  4 Kubernetes            Manifests, Pods,        kubectl output and
                          Services, PVC           application

  5 Ansible               Inventory and playbooks Playbook execution

  6 Terraform             IaC and plan            Plan and cloud
                                                  resources if
                                                  provisioned

  7 Monitoring            Prometheus/Grafana      Live dashboard

  8 Logging               Collector and central   Searchable logs
                          log view                

  9 Security              Code/image scan and     Scan reports and RBAC
                          access controls         

  10 Release              Blue-Green or Canary    Release and rollback
                                                  evidence

  11 GitOps               Argo CD/Flux            Sync and reconciliation
                          configuration           evidence
  -----------------------------------------------------------------------

## 8. Suggested Repository Structure

Adapt this tree to files that exist in the actual repository.

``` text
quickbite/
├── frontend/
├── backend/
├── k8s/
│   ├── namespace.yaml
│   ├── mysql/
│   ├── backend/
│   └── frontend/
├── ansible/
│   ├── ansible.cfg
│   ├── inventory/
│   ├── playbooks/
│   └── roles/
├── terraform/
├── monitoring/
│   ├── prometheus/
│   └── grafana/
├── logging/
├── security/
├── gitops/
├── docs/
│   └── screenshots/
├── docker-compose.yml
├── Jenkinsfile
├── .gitignore
└── README.md
```

## 9. Screenshots and Results

Add screenshots under `docs/screenshots/` or link to an approved
location. Suggested filenames:

``` text
docs/screenshots/
├── 01-application-home.png
├── 02-github-repository.png
├── 03-jenkins-pipeline.png
├── 04-docker-containers.png
├── 05-kubernetes-pods.png
├── 06-kubernetes-services.png
├── 07-ansible-playbook.png
├── 08-terraform-plan.png
├── 09-grafana-dashboard.png
├── 10-centralized-logs.png
├── 11-security-scan.png
├── 12-release-strategy.png
└── 13-gitops-sync.png
```

For every screenshot, caption what it demonstrates, its phase, the
command/pipeline/action that produced it, and whether it is local, cloud
or planned. Never include passwords, tokens, private keys or unmasked
account details.

## 10. Current Implementation Status

Update this table before submission. These are placeholders, not
verified completion claims.

  Phase                     Status            Evidence / notes
  ------------------------- ----------------- --------------------------------
  1\. Git and GitHub        \[In progress\]   \[Add URL\]
  2\. Jenkins CI            \[In progress\]   \[Add build URL/screenshot\]
  3\. Docker                \[In progress\]   \[Add image/Compose evidence\]
  4\. Local Kubernetes      \[In progress\]   \[Add Pod/Service evidence\]
  5\. Ansible               \[Not started\]   \[Add playbook evidence\]
  6\. Terraform/AWS         \[Not started\]   \[Add plan/resource evidence\]
  7\. Monitoring            \[Not started\]   \[Add dashboard evidence\]
  8\. Centralized logging   \[Not started\]   \[Add log evidence\]
  9\. Security/DevSecOps    \[Not started\]   \[Add scan evidence\]
  10\. Blue-Green/Canary    \[Not started\]   \[Add release evidence\]
  11\. GitOps               \[Not started\]   \[Add sync evidence\]

## 11. Conclusion

QuickBite demonstrates how an application can move from source code to a
repeatable deployment and operations workflow. Git and Jenkins provide
source control and continuous integration; Docker packages the
application; Kubernetes manages workloads; Ansible and Terraform
automate configuration and infrastructure; monitoring and logging
improve operational visibility; security checks reduce risk; deployment
strategies support safer releases; and GitOps makes the desired cluster
state traceable in Git.

The final submission should include actual configuration files,
successful command outputs, screenshots and test results for every
completed phase. Clearly distinguish completed local implementation,
cloud resources actually deployed and future learning extensions.

## 12. References

-   Git: https://git-scm.com/doc
-   GitHub Docs: https://docs.github.com/
-   Jenkins: https://www.jenkins.io/doc/
-   Docker: https://docs.docker.com/
-   Kubernetes: https://kubernetes.io/docs/
-   Ansible: https://docs.ansible.com/
-   Terraform: https://developer.hashicorp.com/terraform/docs
-   Prometheus: https://prometheus.io/docs/
-   Grafana: https://grafana.com/docs/
-   SonarQube: https://docs.sonarsource.com/
-   Trivy: https://trivy.dev/
-   Argo CD: https://argo-cd.readthedocs.io/
-   AWS EKS: https://docs.aws.amazon.com/eks/
