# Kubernetes Deployment for HTTPBin

This project provides a minimal, secure, and best-practice Kubernetes deployment of [kennethreitz/httpbin](https://hub.docker.com/r/kennethreitz/httpbin/), a simple HTTP request and response service useful for testing.

---

## Project Structure

```
httpbin-deployment
├── deployment.yaml
└── service.yaml
```

---

## Prerequisites

* Kubernetes cluster (`kubectl` configured)

---

## Deployment

Apply Kubernetes manifests:

```bash
kubectl apply -f deployment.yaml -f service.yaml
```

---

## Verify Deployment

Ensure pods and services are running:

```bash
kubectl get deployment,pods,svc -l app=httpbin
```

Expected output:

```
nestoras@MacBook-Pro ~ % kubectl get deployment,pods,svc -l app=httpbin
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/httpbin   3/3     3            3           10m

NAME                           READY   STATUS    RESTARTS   AGE
pod/httpbin-6956658897-h87mz   1/1     Running   0          5m8s
pod/httpbin-6956658897-nzl62   1/1     Running   0          5m9s
pod/httpbin-6956658897-tp9nx   1/1     Running   0          5m7s

NAME              TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/httpbin   ClusterIP   172.20.44.254   <none>        80/TCP    10m

nestoras@MacBook-Pro httpbin-deployment % kubectl port-forward svc/httpbin 8080:80
nestoras@MacBook-Pro ~ % curl http://localhost:8080/get
{
  "args": {},
  "headers": {
    "Accept": "*/*",
    "Host": "localhost:8080",
    "User-Agent": "curl/8.7.1"
  },
  "origin": "127.0.0.1",
  "url": "http://localhost:8080/get"
}
```

---

## Testing the Service

Test connectivity within the cluster:

```bash
kubectl run test-httpbin --rm -it --image=alpine/curl -- sh
```

Then, inside the container:

```bash
curl http://httpbin
```

You should see a JSON response from HTTPBin.

Alternatively, test locally using port-forwarding:

```bash
kubectl port-forward svc/httpbin 8080:80
curl http://localhost:8080/get
```

---

## Best Practices Implemented

* **Non-root Execution:** Runs as a non-root user for better container security.
* **Custom Gunicorn Port (8080):** The application binds to port 8080 instead of 80, which avoids permission issues with non-root users.
* **Writable `/tmp` Directory:** Mounted a writable `emptyDir` volume at `/tmp` to satisfy Gunicorn's need for temporary file access.
* **Read-only Root Filesystem:** Keeps the root filesystem read-only to minimize the risk surface.
* **Topology Spread Constraints:** Ensures even pod distribution across nodes.
* **Resource Limits:** Prevents resource exhaustion.
* **Security Contexts:**

  * Drops all Linux capabilities.
  * Disables privilege escalation.

## Why RollingUpdate Strategy?
We use the RollingUpdate strategy in the Deployment configuration to ensure zero downtime during application updates. This strategy gradually replaces old Pods with new ones, maintaining application availability at all times. It is ideal for production-grade environments where uninterrupted service is essential.

## Scalability Considerations
The application is intentionally kept simple and lightweight by default to ease development and deployment. However, depending on evolving needs, it can be scaled horizontally by:

* Enabling Horizontal Pod Autoscaler (HPA) to adjust the number of pods dynamically based on CPU or memory usage.
* Integrating with Karpenter for intelligent provisioning of nodes, ensuring cost-efficient and responsive infrastructure scaling. 

---

## Clean-up

Delete deployment and service:

```bash
kubectl delete -f deployment.yaml -f service.yaml
```

---

## Maintainers

* [Nestoras](https://github.com/nestoras)

