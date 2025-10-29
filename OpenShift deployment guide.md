# OpenShift Application Deployment Guide

## Step 1: Install OpenShift CLI (oc)

### Download and Install on Linux

```bash
# Navigate to tmp directory
cd /tmp

# Download the latest version
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/stable/openshift-client-linux.tar.gz

# Extract
tar -xvzf openshift-client-linux.tar.gz

# Move to /usr/local/bin
sudo mv oc kubectl /usr/local/bin/

# Make executable
sudo chmod +x /usr/local/bin/oc /usr/local/bin/kubectl

# Clean up
rm openshift-client-linux.tar.gz README.md
```

### Verify Installation

```bash
oc version
```

---

## Step 2: Connect to OpenShift Cluster

### Via Web Console (Recommended)

1. Open OpenShift Web Console in your browser
2. Click on your name (top right corner)
3. Select **"Copy login command"**
4. Click **"Display Token"**
5. Copy the `oc login --token=... --server=...` command
6. Paste it in your terminal

### Or Direct Login

```bash
oc login https://api.your-cluster.com:6443
```

Enter your username and password when prompted.

---

## Step 3: Working with Projects

### View Available Projects

```bash
# List all projects
oc projects

# Show current project
oc project
```

### Switch to Project

```bash
oc project <project-name>
```

### Create New Project

```bash
oc new-project <your-project-name>
```

---

## Step 4: Create Secret for Private DockerHub

For private images, create a secret with your credentials:

```bash
oc create secret docker-registry dockerhub-secret \
  --docker-server=docker.io \
  --docker-username=<your-username> \
  --docker-password=<your-password> \
  --docker-email=<your-email>
```

Link the secret to the default service account:

```bash
oc secrets link default dockerhub-secret --for=pull
```

---

## Step 5: Deploy Backend Application

### Create Application

```bash
oc new-app --name=<backend-app-name> \
  --docker-image=docker.io/<username>/<image-name>:<tag> \
  --labels=app=<app-label>,component=backend
```

### Configure Port

```bash
oc patch svc/<backend-app-name> -p '{"spec":{"ports":[{"port":<backend-port>,"targetPort":<backend-port>}]}}'
```

Example for port 8000:
```bash
oc patch svc/<backend-app-name> -p '{"spec":{"ports":[{"port":8000,"targetPort":8000}]}}'
```

### Expose Service (Create Route)

```bash
oc expose svc/<backend-app-name>
```

### Verify Deployment

```bash
# Check pods
oc get pods

# Check services
oc get svc

# Check routes
oc get route

# View logs
oc logs -l component=backend
```

---

## Step 6: Deploy Frontend Application

### Create Application

```bash
oc new-app --name=<frontend-app-name> \
  --docker-image=docker.io/<username>/<image-name>:<tag> \
  --labels=app=<app-label>,component=frontend
```

### Configure Port

```bash
oc patch svc/<frontend-app-name> -p '{"spec":{"ports":[{"port":<frontend-port>,"targetPort":<frontend-port>}]}}'
```

Example for port 3000:
```bash
oc patch svc/<frontend-app-name> -p '{"spec":{"ports":[{"port":3000,"targetPort":3000}]}}'
```

### Expose Service

```bash
oc expose svc/<frontend-app-name>
```

### Verify Deployment

```bash
# Check pods
oc get pods

# Check routes
oc get route

# Get frontend URL
oc get route <frontend-app-name> -o jsonpath='{.spec.host}'
```

---

## Step 7: Connect Frontend and Backend

### Option A: Frontend Connects via Internal Service

Backend is accessible within the cluster at:
```
http://<backend-service-name>:<backend-port>
```

Set environment variable:

```bash
oc set env deployment/<frontend-app-name> \
  REACT_APP_BACKEND_URL=http://<backend-service-name>:<backend-port>
```

For other frameworks, adjust the variable name:
```bash
# Vue.js
oc set env deployment/<frontend-app-name> \
  VITE_BACKEND_URL=http://<backend-service-name>:<backend-port>

# Next.js
oc set env deployment/<frontend-app-name> \
  NEXT_PUBLIC_BACKEND_URL=http://<backend-service-name>:<backend-port>
```

### Option B: Frontend Connects via External Route

Get backend URL:

```bash
BACKEND_URL=$(oc get route <backend-app-name> -o jsonpath='https://{.spec.host}')

oc set env deployment/<frontend-app-name> \
  REACT_APP_BACKEND_URL=$BACKEND_URL
```

---

## Step 8: Fix Application Code

### Common Issues to Fix

#### 1. Incorrect API URL Syntax

**Wrong:**
```javascript
const response = axios.post`${BACKEND_URL}/api/endpoint`, data, {
```

**Correct:**
```javascript
const BACKEND_URL = process.env.REACT_APP_BACKEND_URL || 'http://localhost:8000';

const response = await axios.post(`${BACKEND_URL}/api/endpoint`, data, {
  headers: { "Content-Type": "application/json" },
});
```

#### 2. Environment Variables File (.env)

**Wrong:**
```
`{BACKEND_URL}=backend-service`
```

**Correct for React:**
```
REACT_APP_BACKEND_URL=http://<backend-service-name>:<port>
```

**Correct for Vue.js:**
```
VITE_BACKEND_URL=http://<backend-service-name>:<port>
```

**Correct for Next.js:**
```
NEXT_PUBLIC_BACKEND_URL=http://<backend-service-name>:<port>
```

### Rebuild and Push Image

```bash
# On your local machine
docker build -t docker.io/<username>/<image-name>:<tag> .
docker push docker.io/<username>/<image-name>:<tag>
```

### Update in OpenShift

```bash
# Restart deployment
oc rollout restart deployment/<app-name>

# Or force pod recreation
oc delete pod -l component=frontend
```

---

## Step 9: Remove Backend Route (Optional)

If backend should not be publicly accessible:

```bash
# View all routes
oc get route

# Delete backend route
oc delete route <backend-route-name>
```

After this, backend is **only accessible within the cluster**.

---

## Useful Diagnostic Commands

### View Status

```bash
# Project overview
oc status

# All resources
oc get all

# Pods
oc get pods

# Services
oc get svc

# Routes
oc get route

# Deployments
oc get deployment
```

### Logs

```bash
# Logs from specific pod
oc logs <pod-name>

# Logs by label
oc logs -l component=frontend
oc logs -l component=backend

# Follow logs in real-time
oc logs -f <pod-name>

# Last 50 lines
oc logs --tail=50 <pod-name>
```

### Events

```bash
# View events (for troubleshooting)
oc get events --sort-by='.lastTimestamp'
```

### Describe Resources

```bash
# Detailed pod information
oc describe pod <pod-name>

# Deployment information
oc describe deployment <deployment-name>

# Service information
oc describe svc <service-name>
```

### Environment Variables

```bash
# List env variables in deployment
oc set env deployment/<app-name> --list

# Add variable
oc set env deployment/<app-name> NEW_VAR=value

# Remove variable
oc set env deployment/<app-name> NEW_VAR-
```

### Connect to Container

```bash
# SSH into running container
oc rsh <pod-name>

# Execute command in container
oc exec <pod-name> -- ls -la
```

---

## Configure CORS on Backend

If frontend cannot connect to backend due to CORS:

### Python (Flask)

```python
from flask_cors import CORS

app = Flask(__name__)
CORS(app, origins=['https://<frontend-url>'])
```

Or allow all origins (for testing):
```python
CORS(app, origins='*')
```

### Node.js (Express)

```javascript
const cors = require('cors');

app.use(cors({
  origin: 'https://<frontend-url>'
}));
```

Or allow all origins:
```javascript
app.use(cors());
```

### Java (Spring Boot)

```java
@CrossOrigin(origins = "https://<frontend-url>")
@RestController
public class ApiController {
    // ...
}
```

Or globally:
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("https://<frontend-url>");
    }
}
```

---

## Application Architecture

```
┌───────────────────────────────────┐
│   OpenShift Project               │
├───────────────────────────────────┤
│                                   │
│  ┌────────────────────────────┐  │
│  │  Frontend Application      │  │
│  │  Port: <frontend-port>     │  │
│  │  Route: external URL       │  │
│  └──────────┬─────────────────┘  │
│             │                     │
│             │ HTTP requests       │
│             ↓                     │
│  ┌────────────────────────────┐  │
│  │  Backend Application       │  │
│  │  Port: <backend-port>      │  │
│  │  Service: internal only    │  │
│  └────────────────────────────┘  │
│                                   │
└───────────────────────────────────┘
```

---

## Deployment Checklist

- [ ] oc CLI installed
- [ ] Connected to OpenShift cluster
- [ ] Project created
- [ ] DockerHub secret created
- [ ] Backend deployed
- [ ] Frontend deployed
- [ ] Ports configured
- [ ] Routes created
- [ ] Application code fixed
- [ ] Environment variables configured
- [ ] Images rebuilt and pushed
- [ ] CORS configured on backend
- [ ] Application tested

---

## Troubleshooting

### Pod Not Starting

```bash
# Check status
oc get pods

# View events
oc describe pod <pod-name>

# View logs
oc logs <pod-name>
```

Common issues:
- Image pull errors
- Insufficient resources
- Configuration errors
- Port conflicts

### ImagePullBackOff Error

Check:
- Image name is correct
- Secret for private repository exists
- Credentials are valid
- Image exists in registry

```bash
# Verify secret
oc get secret dockerhub-secret

# Check if secret is linked
oc describe sa default
```

### Network Error When Uploading Files

Check:
- Frontend uses correct backend URL
- CORS is properly configured on backend
- Routes and services exist
- Backend is running

```bash
# Check backend route
oc get route <backend-route-name>

# Check backend service
oc get svc <backend-service-name>

# Check backend logs
oc logs -l component=backend
```

### Frontend Not Reading Environment Variables

**Important:** React requires `REACT_APP_` prefix and image rebuild!

Environment variables in React are embedded during build time, not runtime.

Solutions:
1. Rebuild the image with correct variables
2. Use runtime configuration (config.js approach)
3. Use server-side rendering (SSR)

### Pod CrashLoopBackOff

```bash
# View logs
oc logs <pod-name>

# View previous container logs
oc logs <pod-name> --previous

# Describe pod for events
oc describe pod <pod-name>
```

Common causes:
- Application crashes on startup
- Missing environment variables
- Port already in use
- Configuration errors

---

## Advanced Operations

### Scale Application

```bash
# Scale to 3 replicas
oc scale deployment/<app-name> --replicas=3

# View replicas
oc get deployment <app-name>
```

### Update Image

```bash
# Update to new image version
oc set image deployment/<app-name> \
  <container-name>=docker.io/<username>/<image>:<new-tag>

# Check rollout status
oc rollout status deployment/<app-name>
```

### Rollback Deployment

```bash
# View rollout history
oc rollout history deployment/<app-name>

# Rollback to previous version
oc rollout undo deployment/<app-name>

# Rollback to specific revision
oc rollout undo deployment/<app-name> --to-revision=2
```

### Resource Limits

```bash
# Set resource limits
oc set resources deployment/<app-name> \
  --limits=cpu=500m,memory=512Mi \
  --requests=cpu=250m,memory=256Mi
```

### Health Checks

Add to deployment YAML:

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8000
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 8000
  initialDelaySeconds: 5
  periodSeconds: 5
```

Apply:
```bash
oc apply -f deployment.yaml
```

---

## Best Practices

1. **Use specific image tags** instead of `latest`
2. **Set resource limits** to prevent resource exhaustion
3. **Implement health checks** for automatic recovery
4. **Use secrets** for sensitive data
5. **Use ConfigMaps** for configuration
6. **Label resources** consistently
7. **Keep backend internal** (no external route if not needed)
8. **Enable CORS** properly on backend
9. **Use environment-specific configs** (.env files)
10. **Monitor logs and events** regularly

---

## Quick Reference

### Common Commands

```bash
# Login
oc login <cluster-url>

# Switch project
oc project <project-name>

# Create app from image
oc new-app <image-name>

# Expose service
oc expose svc/<service-name>

# View logs
oc logs -f <pod-name>

# Describe resource
oc describe <resource-type> <resource-name>

# Delete resource
oc delete <resource-type> <resource-name>

# Get all resources
oc get all

# Restart deployment
oc rollout restart deployment/<app-name>
```

### Resource Types

- `pod` / `po` - Pods
- `service` / `svc` - Services
- `route` - Routes
- `deployment` / `deploy` - Deployments
- `configmap` / `cm` - ConfigMaps
- `secret` - Secrets
- `persistentvolumeclaim` / `pvc` - Persistent Volume Claims

---

**End of Guide**