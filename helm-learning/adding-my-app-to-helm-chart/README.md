# How to Create a Helm Chart for a Java Application

Here’s a step-by-step guide to package and deploy your Java application on Kubernetes using Helm.

First, ensure you have the **Helm CLI installed** and that your **Java application is containerized** (i.e., you have a Docker image for it pushed to a registry like Docker Hub, GCR, or ACR).

---

### Step 1: Create a Boilerplate Chart 骨架

Helm provides a handy command to create a standard chart structure. Open your terminal and run:

```bash
helm create sample-java-chart
```

This command creates a new directory named `sample-java-chart` with the following file structure:

```
sample-java-chart/
├── Chart.yaml          # Information about your chart
├── charts/             # For chart dependencies
├── templates/          # The directory for your Kubernetes manifest templates
│   ├── NOTES.txt       # Notes displayed after a successful installation
│   ├── _helpers.tpl    # Template helpers
│   ├── deployment.yaml # A template for a Kubernetes Deployment
│   ├── service.yaml    # A template for a Kubernetes Service
│   └── ...
└── values.yaml         # The default configuration values for your chart
```

---

### Step 2: Configure `values.yaml` ⚙️

This is the most important file for customizing your deployment without changing the templates. Open `values.yaml` and modify these key sections:

**1. Image Configuration:**
Change `image.repository` to point to your Java application's image and `image.tag` to the specific version you want to deploy.

```yaml
# values.yaml

image:
  repository: your-docker-registry/sample-java-test # <-- CHANGE THIS
  pullPolicy: IfNotPresent
  tag: "latest" # <-- Or your specific app version, e.g., "1.0.0"
```

**2. Service Configuration:**
Set the `service.port` to the port your Java application listens on (e.g., `8080` for Spring Boot). You can also change the `service.type` to `LoadBalancer` or `NodePort` if you need to access your app from outside the cluster.

```yaml
# values.yaml

service:
  type: ClusterIP # Use LoadBalancer to expose externally
  port: 8080 # <-- Port exposed by the service
```

---

### Step 3: Customize the Deployment Template (Optional) 🛠️

The default templates are often sufficient, but for a Java app, it's a good practice to add **liveness and readiness probes** to your `templates/deployment.yaml`. This helps Kubernetes know if your application is healthy.

Open `templates/deployment.yaml` and add the `livenessProbe` and `readinessProbe` sections inside the `containers` spec. For a Spring Boot app with Actuator, it might look like this:

```yaml
# templates/deployment.yaml

...
spec:
  containers:
    - name: {{ .Chart.Name }}
      image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
      ports:
        - name: http
          containerPort: 8080 # <-- Make sure this matches your app's port
          protocol: TCP
      livenessProbe:
        httpGet:
          path: /actuator/health/liveness
          port: http
      readinessProbe:
        httpGet:
          path: /actuator/health/readiness
          port: http
...
```
> **Note:** Make sure your Java application exposes these health endpoints (e.g., by including the Spring Boot Actuator dependency).

---

### Step 4: Lint and Install Your Chart 🚀

Now you're ready to deploy!

**1. Lint your chart (check for errors):**
Navigate into your chart directory and run `helm lint`. This command checks for syntax errors and best practices.

```bash
cd sample-java-chart
helm lint .
```

**2. Install the chart:**
Use the `helm install` command to deploy your application to your Kubernetes cluster. You need to give your deployment a unique release name (e.g., `my-java-app`).

```bash
# Dry-run first to see the generated Kubernetes YAML
helm install my-java-app . --dry-run --debug

# If it looks good, install it for real!
helm install my-java-app .
```

**3. Check the status:**
You can see the status of your deployment with:

```bash
helm status my-java-app
kubectl get pods
```

---

### Step 5: Troubleshooting Common Errors 🩺

If your pod doesn't say `Running`, use these commands to debug.

**1. Error: `ErrImagePull` or `ImagePullBackOff`**
* **Meaning:** Kubernetes cannot download the Docker image.
* **Diagnosis:** Check the pod events for details.
    ```bash
    kubectl describe pod <your-pod-name>
    ```
* **Common Fixes:**
    * Ensure the `image.repository` in `values.yaml` is correct.
    * Make sure you have run `docker push` to upload your image to the registry.
    * If the repository is private, ensure you've configured an `imagePullSecrets`.

**2. Error: `CrashLoopBackOff`**
* **Meaning:** The container starts but crashes immediately. This is usually an application error.
* **Diagnosis:** Check the container's logs to see the application stack trace or error message.
    ```bash
    kubectl logs <your-pod-name>
    ```
* **Common Fixes:**
    * **Java Version Mismatch:** Check for `UnsupportedClassVersionError` in the logs. Ensure the Java version in your `Dockerfile` matches the version you compiled your code with.
    * Fix the application code or configuration, rebuild and push your Docker image, then upgrade Helm.

**3. Error: Probes are Failing (Pod is `Unhealthy`)**
* **Meaning:** The application is running, but Kubernetes thinks it's not ready to accept traffic.
* **Diagnosis:** Check the pod events. You'll see messages like `Readiness probe failed`.
    ```bash
    kubectl describe pod <your-pod-name>
    ```
* **Common Fixes:**
    * **Port Mismatch:** Ensure the `containerPort` in `deployment.yaml` and the `service.port` in `values.yaml` match the port your application listens on.
    * **Incorrect Health Path:** Ensure the `path` in the `livenessProbe` and `readinessProbe` sections of `deployment.yaml` matches the health endpoint your app exposes (e.g., `/actuator/health`).

**Applying Fixes:**
After fixing your code, `Dockerfile`, or Helm templates, apply the changes by upgrading the release:
```bash
helm upgrade my-java-app .
