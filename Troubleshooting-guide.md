# Phase 3 – Debugging & Issue Resolution

This guide outlines the systematic approach taken to diagnose and resolve three common issues encountered in the e-commerce Kubernetes deployment. Each section covers observed symptoms, diagnostic steps, and the resolution implemented.

## 1. Frontend Not Accessible Externally

**Issue-1**
After deployment, the frontend (React + Nginx) application is not accessible at the expected external URL (via Ingress).

**Diagnosis Steps**

1. Verify the Ingress controller (NGINX) is installed and running in the cluster.
2. Check DNS resolution and confirmed that ecommerce.local resolves to the LoadBalancer IP.
3. Examine Ingress YAML and found correct path/host mappings.
4. Issue can be the frontend service was not correctly labeled for the Ingress selector.
5. kubectl describe ingress -  can show default backend - 404 — indicating service mismatch.

**Resolution**

1. Ensured the frontend service had the correct app: frontend label.
2. Verified the Ingress definition used service.name: frontend correctly.
3. Restarted the NGINX ingress controller to reload configuration.
4. Confirmed success by accessing http://ecommerce.local via port-forward and public LB.


**Issue-2**
Node.js backend microservices (user, order, inventory) were intermittently losing connection to MongoDB, causing ECONNRESET and timeout errors.

**Diagnosis Steps**
1. Check MongoDB logs (kubectl logs) — no crash, but sporadic connection spikes.
2. Backend logs shows frequent retries and timeouts.
3. Verify service DNS resolution: nslookup mongodb.default.svc.cluster.local
4. NetworkPolicy can be too restrictive (didn’t allow backend traffic to MongoDB).
5. Check readinessProbes for MongoDB — passed.

**Resolution**
1. Modified NetworkPolicy to allow ingress from backend pods with app=user|order|inventory labels.
2. Increased MongoDB connection pool settings in backend ConfigMap.
3. Applied graceful error handling and retry logic with backoff.
4. After changes, no connection resets observed under load testing.

**Issue-3**
Orders placed via frontend were significantly delayed in processing. Backend queue consumers seemed stuck intermittently.

**Diagnosis Steps**
1. Verify RabbitMQ pod health — running and reachable.
2. Logged into RabbitMQ dashboard (:15672) to inspect queues.
3. Can be growing number of messages in orderQueue with low consumer throughput.
4. Confirmed consumer service was deployed — but logs showed slow message acknowledgments.
5. No resource limits on RabbitMQ or consumer pods — leading to saturation under load.


**Resolution**
1. Added resource requests and limits to RabbitMQ and backend consumers.
2. Enabled prefetch and concurrency tuning in RabbitMQ client libraries (prefetch=10).
3. Scaled consumer pods horizontally (via HPA).
4. Queue backlog cleared after tuning — order processing latency dropped below 1s.