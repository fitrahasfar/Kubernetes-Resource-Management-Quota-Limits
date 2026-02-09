
# Kubernetes Resource Management: Quota & Limits

This repository contains configuration files to manage resource consumption (CPU, Memory, Storage) and object counts within a Kubernetes Namespace. These policies help prevent "noisy neighbor" issues and ensure fair resource distribution.

## 📂 Configuration Files

The setup consists of two main components:

1.  **`resourcesquota.yml`** (ResourceQuota): Defines the **TOTAL** aggregate resource limits for the entire Namespace.
2.  **`limitrange.yml`** (LimitRange): Defines the **PER-ITEM** constraints (validations and defaults) for Containers and PVCs.

---

## 1. ResourceQuota (`rs-go-api`)

File: `resourcesquota.yml`

This object acts as the "Total Budget" for the Namespace. The sum of all active resources cannot exceed these values.

### Object Count Limits
| Object Type | Maximum Allowed |
| :--- | :--- |
| **Pods** | 5 |
| **Services** | 5 |
| **ConfigMaps** | 3 |
| **Secrets** | 2 |
| **PVCs** | 3 |

### Compute Resources (Namespace Total)
The combined total of all running Pods must stay within:

| Resource | Requests (Guaranteed) | Limits (Hard Cap) |
| :--- | :--- | :--- |
| **CPU** | `500m` (0.5 Core) | `1` (1 Core) |
| **Memory** | `512Mi` | `1Gi` |
| **Storage** | `10Gi` (Total sum of all PVC requests) | - |

> ⚠️ **Note:** If you attempt to create a 6th Pod or exceed the 500m CPU request threshold, Kubernetes will reject the deployment.

---

## 2. LimitRange (`limit-range`)

File: `limitrange.yml`

This object acts as a "Gatekeeper & Auto-filler" for **individual containers** or **individual PVCs**.

### Container Constraints (CPU & Memory)
Applied to every container within a Pod.

| Type | CPU | Memory | Description |
| :--- | :--- | :--- | :--- |
| **Min** | `50m` | `128Mi` | Pod **rejected** if request is lower than this. |
| **Max** | `1` | `2Gi` | Pod **rejected** if limit is higher than this. |
| **Default Request** | `50m` | `128Mi` | Applied automatically if `requests` is not specified. |
| **Default Limit** | `100m` | `256Mi` | Applied automatically if `limits` is not specified. |

### Storage Constraints (PVC)
Applied to every PersistentVolumeClaim creation.

| Type | Storage |
| :--- | :--- |
| **Min** | `1Gi` |
| **Max** | `10Gi` |

---

## 🚀 How to Apply

1. **Apply ResourceQuota:**
   ```bash
   kubectl apply -f resourcesquota.yml

```

2. **Apply LimitRange:**
```bash
kubectl apply -f limitrange.yml

```


3. **Verification:**
Check current quota usage (Used vs Hard):
```bash
kubectl get resourcequota rs-go-api --output=yaml

```


Check limit range rules:
```bash
kubectl get limitrange limit-range --output=yaml

```



## 🧪 Testing Scenarios

* **Auto-fill Test:** Deploy a Pod without any resource definitions. It will automatically be assigned 100m CPU / 256Mi RAM (from `default` settings).
* **Min-Validation Failure:** Deploy a Pod with 64Mi Memory request. It will be **rejected** because it's below the 128Mi minimum.
* **Quota Failure:** Attempt to deploy more than 5 Pods. The 6th Pod will be **rejected** by the ResourceQuota.

```

***
3. **Professional Tone:** Sangat cocok jika Anda ingin memamerkan kode ini di portofolio GitHub atau dibaca oleh rekan kerja internasional.

```
