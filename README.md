# Kubernetes YAML Troubleshooting Worksheet
## From Symptom → Signal → Root Cause

---

### Purpose

This worksheet is designed to help you **systematically diagnose Kubernetes issues**
that originate from **misconfigured YAML files**.

You are expected to think, observe, and reason.

---

## Ground Rules

- You start from a **working baseline**
- In each exercise, **one YAML file is faulty**
- Do NOT jump directly to editing YAML
- Follow the steps in order

> Kubernetes is declarative.  
> If something breaks — **a contract between resources is broken**.

---

## Allowed Diagnostic Tools

Use the following:

````md
oc expose svc service-a
<ROUTE URL>/docs # to test endpoints

kubectl get
kubectl describe
kubectl logs
kubectl get endpoints
kubectl exec
kubectl get events


````

---

# Troubleshooting Template  
(Repeat this section for each broken YAML)

---

## Case #: _______________________________

### 1. Initial Observation



**Observed symptom (check all that apply):**

- [ ] Pod not created
- [ ] Pod stuck in `Pending`
- [ ] Pod in `CrashLoopBackOff`
- [ ] Pod is `Running` but service not reachable
- [ ] `kubectl apply` failed
- [ ] Other: _______________________________

---

### 2. First System Snapshot

Command executed:

```

oc get all -l lab=ip-lab

```

**What looks suspicious?**

| Resource | Status | Why it looks suspicious |
|--------|--------|-------------------------|
|        |        |                         |
|        |        |                         |

---

### 3. Signal Collection

Which command did you run next?

```

---

```

**Key output / message (copy only the important part):**

```

---

---

---

```

---

### 4. Reasoning (Critical Step)

Answer in full sentences:

**What relationship or contract between Kubernetes resources seems broken?**

Examples (do NOT copy):
- Service ↔ Pod labels
- Service port ↔ containerPort
- DNS name ↔ Service name
- VolumeMount ↔ PVC

Your reasoning:

```

---

---

---

```

---

### 5. YAML Inspection

Which area of the YAML do you suspect?

- [ ] metadata.labels
- [ ] spec.selector
- [ ] spec.template.metadata.labels
- [ ] Service port / targetPort
- [ ] containerPort
- [ ] Environment variable value
- [ ] spec.serviceName (StatefulSet)
- [ ] volumeMounts.name
- [ ] volumeClaimTemplates.name
- [ ] Other: _______________________________

**Exact field path:**

```

spec.________________________________________

```

---

### 6. Fix Description (No YAML)

Describe the fix **in words only**:

> “I changed ________ to ________.”

```

---

---

```

---

### 7. Validation

After applying the fix:

**What changed? (check all that apply)**

- [ ] Pod status improved
- [ ] Endpoints appeared
- [ ] CrashLoop stopped
- [ ] Logs show healthy startup
- [ ] Service reachable

**Proof command used:**

```

kubectl ______________________________________

```

### 8. Cleanup

Don't forget to clean up your workspace:

```
oc delete all -l lab=ip-lab
oc delete pvc -l lab=ip-lab
```


# Hints:
* Compare **pod labels** vs **service selector**
* Commands:
```
  oc get pods --show-labels
  oc describe svc <service name>
  oc get endpoints <service name>
```