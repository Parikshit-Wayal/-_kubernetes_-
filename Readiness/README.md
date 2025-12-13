🔹 Step 1: What is a Readiness Probe?

A Readiness Probe is a health check used by Kubernetes to decide:

👉 Should this pod receive traffic or not?

If readiness probe passes → pod receives traffic

If readiness probe fails → pod is removed from service traffic

📌 Important:
Readiness probe does NOT restart the container.

🔹 Step 2: Why Do We Need a Readiness Probe?

In real applications:

App may start, but DB is not connected

App may become temporarily unavailable

App may be overloaded

In these cases:

App should stay running

But should NOT receive traffic

👉 This is exactly what the readiness probe solves.

🔹 Step 3: Pod Creation and Initial State

When the deployment is applied, the pod starts running.

kubectl get pods


At the beginning, the pod may show 0/1 READY because readiness probe has not passed yet.

<img width="937" height="250" alt="pod" src="https://github.com/user-attachments/assets/00227f20-4e37-4ce8-a48b-f3e6cc5ad9ac" />

📌 Meaning:

Pod is Running

But it is NOT ready to receive traffic

🔹 Step 4: Readiness Probe Endpoint in Application

We created a /ready endpoint in the application.

Behavior:

First 30 seconds → app is READY

After 30 seconds → app becomes NOT READY

Code logic (simple meaning):

If app runtime < 30 sec → return 200 (READY)

If app runtime > 30 sec → return 503 (NOT READY)

📌 This simulates a real dependency failure (like DB down).

🔹 Step 5: Readiness Probe in Kubernetes YAML

Kubernetes continuously calls the /ready endpoint.

What Kubernetes does:

Calls /ready every 5 seconds

If response is 200 → pod is marked READY

If response is 503 → pod is marked NOT READY

📌 Kubernetes does not kill the pod, it only controls traffic.

🔹 Step 6: Service Creation

A Kubernetes Service is created to expose the pod.

kubectl get svc

<img width="722" height="183" alt="service" src="https://github.com/user-attachments/assets/8736b16b-8d79-4b07-966a-5d944090e7be" />

📌 Service sends traffic only to READY pods.

🔹 Step 7: Endpoints Behavior (Most Important Concept)

You monitored service endpoints:

kubectl get endpoints readiness-service -w

<img width="558" height="168" alt="endpoint" src="https://github.com/user-attachments/assets/7c7f909e-16c1-4fe4-8f72-b89a4d5d5a91" />
Meaning:

<none> → No pod is ready → no traffic

Pod IP appears → Pod is ready → traffic allowed

📌 Readiness probe directly controls service endpoints.

🔹 Step 8: Events Showing Readiness Probe Failure

You checked pod events:

kubectl describe pod <pod-name>

<img width="949" height="221" alt="events" src="https://github.com/user-attachments/assets/df0ebf65-8136-4f3b-9f03-5153028f1da3" />

You saw:

Readiness probe failed: HTTP probe failed with statuscode 503


📌 Important:

❌ Container is NOT restarted

❌ Pod is NOT deleted

✅ Only traffic is stopped

🔹 Step 9: Pod Status Change (Key Observation)

You ran:

kubectl get pods -w

<img width="632" height="193" alt="running" src="https://github.com/user-attachments/assets/6904405c-8641-4474-b0ca-3ee541bc7161" />
Output:
READY   STATUS
0/1     Running
1/1     Running


📌 Meaning:

Running → container is alive

0/1 → pod is NOT receiving traffic

1/1 → pod is receiving traffic

🔹 Step 10: Traffic Test Using Curl

You tested traffic via the service.

When Readiness FAILED:
curl readiness-service

<img width="704" height="200" alt="failes to connect" src="https://github.com/user-attachments/assets/8e8d10c5-3d81-478d-bbd0-be36eaabee90" />

📌 Reason:

Pod was NOT ready

Service had no endpoints

Traffic was blocked

When Readiness PASSED:
curl readiness-service


Response:

App is running


📌 Traffic works because pod is READY.

🔹 Step 11: Final Flow Summary
When Readiness Probe FAILS:

Pod status → Running

READY → 0/1

Endpoints → none

Traffic → ❌ stopped

Restart → ❌ NO

When Readiness Probe PASSES:

Pod status → Running

READY → 1/1

Endpoints → Pod IP

Traffic → ✅ allowed

🔹 Final One-Line Summary (Documentation Ready)

Readiness probe checks whether a pod is ready to receive traffic. If it fails, Kubernetes removes the pod from service endpoints without restarting it, ensuring traffic is routed only to ready and healthy pods.

✅ Conclusion

Readiness probe working correctly

Traffic controlled properly

No container restart

Real production-like behavior

If you want next, I can document Startup Probe or All 3 Probes Together in the same format 👍
