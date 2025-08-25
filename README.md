# Kubernetes Manifests + Prompt Portfolio

> This repo contains a portfolio of concise prompts (for `kubectl-ai` or any LLM) that generate and analyze Kubernetes YAML for a demo app.  
> Output policy: **return valid Kubernetes YAML only**, separated by `---` when multiple objects are needed.

> Tooling: [kubectl-ai](https://github.com/GoogleCloudPlatform/kubectl-ai)  
> Reference: Google’s *Prompt Engineering* whitepaper (Kaggle rehost)

## Table

| NAME | PROMPT | DESCRIPTION | EXAMPLE |
|---|---|---|---|
| app.yaml | Generate a Deployment and ClusterIP Service for app `go-demo-app` in namespace `demo`. Image `paranoidlookup/demo-app:v1.0.3`. Expose containerPort 8080; Service port 80 → targetPort 8080. 2 replicas. Add labels `app=go-demo-app`. Return ONLY valid Kubernetes YAML. | Base app + stable service wiring. | [./yaml/app.yaml](./yaml/app.yaml) |
| app-livenessProbe.yaml | Generate a Deployment `go-demo-app` (ns=`demo`) with a `livenessProbe` `httpGet` on path `/healthz`, port 8080, `initialDelaySeconds: 10`, `periodSeconds: 5`, `failureThreshold: 3`. YAML only. | Liveness check for self-heal. | [./yaml/app-livenessProbe.yaml](./yaml/app-livenessProbe.yaml) |
| app-readinessProbe.yaml | Generate a Deployment `go-demo-app` (ns=`demo`) with a `readinessProbe` `httpGet` on `/readyz`, port 8080, `initialDelaySeconds: 5`, `periodSeconds: 5`, `successThreshold: 1`, `failureThreshold: 3`. Image `paranoidlookup/demo-app:v1.0.3`. YAML only. | Readiness gate for traffic. | [./yaml/app-readinessProbe.yaml](./yaml/app-readinessProbe.yaml) |
| app-volumeMounts.yaml | Generate a Deployment `go-demo-app` (ns=`demo`) that mounts a ConfigMap `app-config` to `/etc/app` (readOnly) and an `emptyDir` to `/var/cache/app`. Include the ConfigMap with key `config.yml: "enable=true"`. Image `paranoidlookup/demo-app:v1.0.3`. YAML only. | Config + runtime cache mounts. | [./yaml/app-volumeMounts.yaml](./yaml/app-volumeMounts.yaml) |
| app-cronjob.yaml | Generate a CronJob `db-backup` (ns=`demo`) schedule `0 2 * * *` that runs `alpine:3` with `sh -c "echo backup && date"`. Set `concurrencyPolicy: Forbid`, `successfulJobsHistoryLimit: 3`, `failedJobsHistoryLimit: 1`, `ttlSecondsAfterFinished: 3600`. YAML only. | Nightly maintenance task. | [./yaml/app-cronjob.yaml](./yaml/app-cronjob.yaml) |
| app-job.yaml | Generate a one-off Job `db-migrate` (ns=`demo`) using `alpine:3`, command `sh -c "echo migrate && exit 0"`, `restartPolicy: Never`, `backoffLimit: 2`. YAML only. | One-off job/migration. | [./yaml/app-job.yaml](./yaml/app-job.yaml) |
| app-multicontainer.yaml | Generate a Deployment `go-demo-app` (ns=`demo`) with two containers: (1) app `paranoidlookup/demo-app:v1.0.3` on 8080; (2) sidecar `nginx:1.27` on 80 → 127.0.0.1:8080 via ConfigMap `nginx.conf`. Add a ClusterIP Service on port 80 to nginx. YAML only. | Sidecar reverse proxy pattern. | [./yaml/app-multicontainer.yaml](./yaml/app-multicontainer.yaml) |
| app-resources.yaml | Generate a Deployment `go-demo-app` (ns=`demo`) with resource requests/limits: cpu `100m/500m`, memory `128Mi/512Mi`. 2 replicas. YAML only. | Right-sized resources. | [./yaml/app-resources.yaml](./yaml/app-resources.yaml) |
| app-secret-env.yaml | Generate a Secret `app-secrets` (ns=`demo`) with `stringData: { API_KEY: "changeme", DB_PASSWORD: "changeme" }` and a Deployment `go-demo-app` that reads them via `env.valueFrom.secretKeyRef`. YAML only. | Secrets → env injection. | [./yaml/app-secret-env.yaml](./yaml/app-secret-env.yaml) |

---