# Kubernetes Manifests + Prompt Portfolio

> This repo contains a portfolio of concise prompts (for `kubectl-ai` or any LLM) that generate and analyze Kubernetes YAML for a demo app.  
> Output policy: **return valid Kubernetes YAML only**, separated by `---` when multiple objects are needed.

> Tooling: [kubectl-ai](https://github.com/GoogleCloudPlatform/kubectl-ai)  
> Reference: Google’s *Prompt Engineering* whitepaper (Kaggle rehost)

## Table
markdown
| NAME | PROMPT | DESCRIPTION | EXAMPLE |
|---|---|---|---|
| app.yaml | **Generate a Deployment and ClusterIP Service for app `go-demo-app` in namespace `demo`. Image `paranoidlookup/demo-app:v1.0.3`. Expose containerPort 8080; Service port 80 → targetPort 8080. 2 replicas. Add labels `app=go-demo-app`. Return ONLY valid Kubernetes YAML.** | Base app + stable service wiring. | [/yaml/app.yaml](./yaml/app.yaml) |
| app-livenessProbe.yaml | **Generate a Deployment `go-demo-app` (ns=`demo`) with a `livenessProbe` `httpGet` on path `/healthz`, port 8080, `initialDelaySeconds: 10`, `periodSeconds: 5`, `failureThreshold: 3`. Keep 1 container using `paranoidlookup/demo-app:v1.0.3`. YAML only.** | Adds a robust liveness check. | [/yaml/app-livenessProbe.yaml](./yaml/app-livenessProbe.yaml) |
| app-readinessProbe.yaml | **Generate a Deployment `go-demo-app` (ns=`demo`) with a `readinessProbe` `httpGet` on `/readyz`, port 8080, `initialDelaySeconds: 5`, `periodSeconds: 5`, `successThreshold: 1`, `failureThreshold: 3`. Image `paranoidlookup/demo-app:v1.0.3`. YAML only.** | Adds a readiness gate for traffic. | [/yaml/app-readinessProbe.yaml](./yaml/app-readinessProbe.yaml) |
| app-volumeMounts.yaml | **Generate a Deployment `go-demo-app` (ns=`demo`) that mounts a ConfigMap `app-config` to `/etc/app` (readOnly) and an `emptyDir` to `/var/cache/app`. Include the ConfigMap with key `config.yml: "enable=true"`. Image `paranoidlookup/demo-app:v1.0.3`. YAML only.** | Shows config and runtime cache mounts. | [/yaml/app-volumeMounts.yaml](./yaml/app-volumeMounts.yaml) |
| app-cronjob.yaml | **Generate a CronJob `db-backup` (ns=`demo`) schedule `0 2 * * *` that runs `alpine:3` with `sh -c 'echo backup && date'`. Set `concurrencyPolicy: Forbid`, `successfulJobsHistoryLimit: 3`, `failedJobsHistoryLimit: 1`, `ttlSecondsAfterFinished: 3600`. YAML only.** | Nightly backup-style task. | [/yaml/app-cronjob.yaml](./yaml/app-cronjob.yaml) |
| app-job.yaml | **Generate a one-off Job `db-migrate` (ns=`demo`) using `alpine:3`, command `sh -c 'echo migrate && exit 0'`, `restartPolicy: Never`, `backoffLimit: 2`. YAML only.** | Simple migration job. | [/yaml/app-job.yaml](./yaml/app-job.yaml) |
| app-multicontainer.yaml | **Generate a Deployment `go-demo-app` (ns=`demo`) with two containers: (1) app `paranoidlookup/demo-app:v1.0.3` on 8080; (2) sidecar `nginx:1.27` reverse proxy on 80 → 127.0.0.1:8080 using a ConfigMap-provided `nginx.conf`. Add a ClusterIP Service on port 80 to the nginx container. YAML only.** | Sidecar pattern (reverse proxy). | [/yaml/app-multicontainer.yaml](./yaml/app-multicontainer.yaml) |
| app-resources.yaml | **Generate a Deployment `go-demo-app` (ns=`demo`) with resource requests/limits: cpu `100m/500m`, memory `128Mi/512Mi`. Image `paranoidlookup/demo-app:v1.0.3`. 2 replicas. YAML only.** | Right-size resources. | [/yaml/app-resources.yaml](./yaml/app-resources.yaml) |
| app-secret-env.yaml | **Generate a Secret `app-secrets` (ns=`demo`) with `stringData: { API_KEY: "changeme", DB_PASSWORD: "changeme" }` and a Deployment `go-demo-app` that reads them via `env.valueFrom.secretKeyRef`. Image `paranoidlookup/demo-app:v1.0.3`. YAML only.** | Secrets → env injection. | [/yaml/app-secret-env.yaml](./yaml/app-secret-env.yaml) |

---

## How to reproduce manifests with `kubectl-ai`

```bash
# Example: generate base app.yaml directly
kubectl ai -p 'Generate a Deployment and ClusterIP Service for app `go-demo-app` in namespace `demo`. Image `paranoidlookup/demo-app:v1.0.3`. Expose containerPort 8080; Service port 80 → targetPort 8080. 2 replicas. Add labels `app=go-demo-app`. Return ONLY valid Kubernetes YAML.' > yaml/app.yaml
```


---

### `yaml/app.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-demo-app
  namespace: demo
  labels:
    app: go-demo-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: go-demo-app
  template:
    metadata:
      labels:
        app: go-demo-app
    spec:
      containers:
        - name: app
          image: paranoidlookup/demo-app:v1.0.3
          ports:
            - containerPort: 8080
          env:
            - name: VERSION
              value: "v1"
---
apiVersion: v1
kind: Service
metadata:
  name: go-demo-app
  namespace: demo
  labels:
    app: go-demo-app
spec:
  type: ClusterIP
  selector:
    app: go-demo-app
  ports:
    - name: http
      port: 80
      targetPort: 8080
```

### yaml/app-livenessProbe.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-demo-app
  namespace: demo
  labels:
    app: go-demo-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: go-demo-app
  template:
    metadata:
      labels:
        app: go-demo-app
    spec:
      containers:
        - name: app
          image: paranoidlookup/demo-app:v1.0.3
          ports:
            - containerPort: 8080
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 5
            failureThreshold: 3
```
### yaml/app-readinessProbe.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-demo-app
  namespace: demo
  labels:
    app: go-demo-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: go-demo-app
  template:
    metadata:
      labels:
        app: go-demo-app
    spec:
      containers:
        - name: app
          image: paranoidlookup/demo-app:v1.0.3
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              path: /readyz
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 5
            successThreshold: 1
            failureThreshold: 3
```
### yaml/app-volumeMounts.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: demo
data:
  config.yml: |
    enable: true
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-demo-app
  namespace: demo
  labels:
    app: go-demo-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: go-demo-app
  template:
    metadata:
      labels:
        app: go-demo-app
    spec:
      volumes:
        - name: cfg
          configMap:
            name: app-config
        - name: cache
          emptyDir: {}
      containers:
        - name: app
          image: paranoidlookup/demo-app:v1.0.3
          ports:
            - containerPort: 8080
          volumeMounts:
            - name: cfg
              mountPath: /etc/app
              readOnly: true
            - name: cache
              mountPath: /var/cache/app
```
### yaml/app-cronjob.yaml
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: db-backup
  namespace: demo
spec:
  schedule: "0 2 * * *"
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      ttlSecondsAfterFinished: 3600
      template:
        spec:
          restartPolicy: OnFailure
          containers:
            - name: backup
              image: alpine:3
              command: ["sh", "-c", "echo backup && date"]
```
### yaml/app-job.yaml
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migrate
  namespace: demo
spec:
  backoffLimit: 2
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: migrate
          image: alpine:3
          command: ["sh", "-c", "echo migrate && exit 0"]
```
### yaml/app-multicontainer.yaml
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-proxy-conf
  namespace: demo
data:
  nginx.conf: |
    events {}
    http {
      server {
        listen 80;
        location / {
          proxy_pass http://127.0.0.1:8080;
          proxy_set_header Host $host;
        }
      }
    }
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-demo-app
  namespace: demo
  labels:
    app: go-demo-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: go-demo-app
  template:
    metadata:
      labels:
        app: go-demo-app
    spec:
      volumes:
        - name: nginx-conf
          configMap:
            name: nginx-proxy-conf
            items:
              - key: nginx.conf
                path: nginx.conf
      containers:
        - name: app
          image: paranoidlookup/demo-app:v1.0.3
          ports:
            - containerPort: 8080
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
          volumeMounts:
            - name: nginx-conf
              mountPath: /etc/nginx/nginx.conf
              subPath: nginx.conf
---
apiVersion: v1
kind: Service
metadata:
  name: go-demo-app
  namespace: demo
  labels:
    app: go-demo-app
spec:
  type: ClusterIP
  selector:
    app: go-demo-app
  ports:
    - name: http
      port: 80
      targetPort: 80
```
### yaml/app-resources.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-demo-app
  namespace: demo
  labels:
    app: go-demo-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: go-demo-app
  template:
    metadata:
      labels:
        app: go-demo-app
    spec:
      containers:
        - name: app
          image: paranoidlookup/demo-app:v1.0.3
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 512Mi
```
### yaml/app-secret-env.yaml
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
  namespace: demo
type: Opaque
stringData:
  API_KEY: "changeme"
  DB_PASSWORD: "changeme"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: go-demo-app
  namespace: demo
  labels:
    app: go-demo-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: go-demo-app
  template:
    metadata:
      labels:
        app: go-demo-app
    spec:
      containers:
        - name: app
          image: paranoidlookup/demo-app:v1.0.3
          ports:
            - containerPort: 8080
          env:
            - name: API_KEY
              valueFrom:
                secretKeyRef:
                  name: app-secrets
                  key: API_KEY
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: app-secrets
                  key: DB_PASSWORD
```