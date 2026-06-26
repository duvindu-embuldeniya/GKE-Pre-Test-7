# What is NEG

**NEG (Network Endpoint Group)** is a list that the Google Cloud Load Balancer uses to know exactly which pod IPs to send traffic to.

## Normal Flow

```
User Browser → GCE Load Balancer → Ingress → Service → Pod
```

This is the flow most people assume that traffic goes through the Service like it does for internal cluster calls.

## Real Flow (what actually happens with NEG)

```
User Browser → GCE Load Balancer → NEG → Pod (direct)
```

- **Ingress** → only used at setup time, tells GKE to provision the Load Balancer and how to route to it.
- **Service** → only used at setup time, tells GKE which pods (by selector) belong in the NEG.
- **NEG** → the actual live backend list the Load Balancer uses to send traffic.

Service and Ingress are bypassed for the real external traffic path, only the NEG is live.

## The GKE/GCE Limitation

When a Service's selector changes (e.g. blue → green during a deployment switch) the new pods aren't in the NEG yet, they only get added after the selector change is detected.

```
Patch Service selector
      ↓
GKE notices the selector changed
      ↓
NEG controller updates: removes old pods, adds new pods
      ↓
Update propagates to the Load Balancer
      ↓
Traffic flows correctly again
```
