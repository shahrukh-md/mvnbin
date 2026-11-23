# Kubernetes (Minikube) & Nagios (Week 10)

## Concepts
- **Kubernetes (K8s):** automates running and managing containerised apps. Docker only runs containers; if one fails, Kubernetes recreates it.
- **Pod:** smallest deployable unit (one or more containers).
- **Deployment:** manages a set of identical pods (replicas, rolling updates, rollbacks).
- **Service:** stable network endpoint that exposes pods (ClusterIP / NodePort / LoadBalancer).
- **Minikube:** runs a single-node local Kubernetes cluster. **kubectl:** CLI to talk to the cluster.
- **Nagios:** open-source monitoring tool for servers, network devices, applications and services; alerts admins when problems occur and when they're resolved.

---

## A. Install Minikube (Windows)

**Prerequisites:** virtualization enabled in BIOS; a driver — Docker Desktop (easiest), Hyper-V or VirtualBox.

1. Open PowerShell **as administrator**.
2. Download + install:
```powershell
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-installer.exe
.\minikube-installer.exe
```
3. If needed add the install folder (e.g. `C:\Program Files\Minikube`) to **PATH** (System Properties → Environment Variables).
4. Start the cluster:
```bash
minikube start --driver=docker       # or --driver=hyperv
minikube status
```
5. Verify:
```bash
minikube version
kubectl version --client
minikube kubectl -- get pods -A
minikube dashboard                    # optional web UI
```
**Troubleshooting:** make sure the driver is running; `minikube logs`; `minikube delete` then `minikube start`.

---

## B. Create, expose and scale pods (Nginx)

```bash
minikube start

# 1. create deployment
kubectl create deployment mynginx --image=nginx
#   (already exists? kubectl set image deployment/mynginx nginx=nginx:latest)
kubectl get deployments
kubectl get pods
kubectl describe pods

# 2. expose as a service
kubectl expose deployment mynginx --type=NodePort --port=80 --target-port=80
kubectl get service mynginx

# 3. scale to 4 pods
kubectl scale deployment mynginx --replicas=4
kubectl get pods

# 4. access the app
kubectl port-forward svc/mynginx 8081:80          # then open http://localhost:8081
#   if error, use:
minikube tunnel
minikube service mynginx --url                    # open the printed URL
minikube dashboard

# 5. clean up
kubectl delete deployment mynginx
kubectl delete service mynginx
minikube stop        # optional
minikube delete      # optional: remove cluster
```
(Manual has a typo `myngnix` — the service name is `mynginx`.)

---

## C. Nagios on Docker (port 8888)

```bash
docker pull jasonrivers/nagios:latest
docker run --name nagiosdemo -p 8888:80 jasonrivers/nagios:latest
```
Open **http://localhost:8888** → login `nagiosadmin` / `nagios` (as in your manual).

Explore:
- **Hosts:** systems being monitored (status **UP/DOWN**).
- **Services:** what's monitored on each host (CPU load, disk, HTTP …).
- **Alerts / Notifications:** recent problems.
- **Host details:** select a host → status, metrics (CPU, memory, network), actions (reschedule check, disable notifications, schedule downtime).

Start/stop/clean:
```bash
docker stop nagiosdemo
docker start nagiosdemo
docker logs -f nagiosdemo          # real-time logs
docker rm nagiosdemo
docker images
docker rmi jasonrivers/nagios:latest
```
Also observe the running container (port 8888) in Docker Desktop / Docker Hub.

---

## D. AWS Free-Tier account creation

1. Open https://aws.amazon.com → **Create an AWS account**.
2. Enter e-mail ID and account name → enter the **verification code** received by mail.
3. Provide contact details, accept terms → **Create / Continue**.
4. Provide **billing details** (card) → **Verify and continue**. (A small amount, e.g. ₹2, is charged and refunded after verification.)
5. Enter phone number (select country) → **Send SMS** → enter the code received.
6. Choose **Basic (free) support plan** → **Complete sign-up**.
7. Click **AWS Management Console** → select role **Academic/Researcher**, interest **DevOps** → Submit.
8. Sign in as **Root user** → the AWS console appears.

---

## E. Scenario answers

1. **Pod keeps restarting (CrashLoopBackOff):**
```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl logs <pod-name>            # add --previous for the crashed container
```
Look for wrong command, missing config/env, failing probes, OOMKilled, bad image.
2. **Pod stuck in Pending:** `kubectl describe pod <pod>` (Events section) and `kubectl get nodes`. Causes: insufficient CPU/memory, no node matches selector/taints, unbound PersistentVolumeClaim, image pull problem.
3. **Debug failed deployment:**
```bash
kubectl get deployments
kubectl describe deployment <name>
kubectl get pods
kubectl get events --sort-by=.metadata.creationTimestamp
```
4. **Pods failing health checks:** `kubectl describe pod` and `kubectl logs`; check the health endpoint, port, startup time, readiness/liveness probe settings (path, port, `initialDelaySeconds`); fix and confirm pods become **Ready**.
5. **Roll back:**
```bash
kubectl rollout history deployment/<name>
kubectl rollout undo deployment/<name>
kubectl rollout status deployment/<name>
```
6. **Debug a running pod:** `kubectl describe pod <pod>`; `kubectl logs <pod>`; `kubectl exec -it <pod> -- sh`.
7. **Expose a local service:** `kubectl port-forward svc/mynginx 8081:80` (or `minikube service mynginx --url`).
8. **Start/stop Nagios:** `docker start nagiosdemo` / `docker stop nagiosdemo`.
9. **"Unable to connect to Nagios process":** `docker ps -a` (is it running?), `docker logs nagiosdemo`; restart the container; fix config errors reported.
10. **New host not showing:** verify the host config file exists and is included in the active config, validate syntax (`nagios -v nagios.cfg`), reload/restart Nagios.
11. **Is Nagios running?** `docker ps`, `docker logs nagiosdemo`, open the web UI.
12. **Real-time logs:** `docker logs -f nagiosdemo`.
13. **Advantages of Nagios:** monitors infrastructure, hosts and services; detects failures and performance problems early; sends alerts when checks fail and when they recover; open source, extensible with plugins.
