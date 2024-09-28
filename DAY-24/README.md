
# Kubernetes ClusterRole and ClusterRoleBinding for Node Access

This guide explains how to create a `ClusterRole` and `ClusterRoleBinding` to grant a user read-only access to nodes across a Kubernetes cluster, both through YAML manifests and directly using `kubectl` commands.

## Steps

### 1. Create a ClusterRole for Node Read Permissions

To grant the user permission to list, get, and watch nodes in the cluster, you can either create a `ClusterRole` using a YAML manifest or directly via the command line.

#### Option 1: Using a YAML Manifest

Create a YAML file named `node-reader-clusterrole.yaml` with the following content:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "watch", "list"]
```

Apply the `ClusterRole`:

```bash
kubectl apply -f node-reader-clusterrole.yaml
```

#### Option 2: Using a Direct `kubectl` Command

Alternatively, you can create the `ClusterRole` using a single command:

```bash
kubectl create clusterrole node-reader \
  --verb=get --verb=watch --verb=list \
  --resource=nodes
```

### 2. Create a ClusterRoleBinding to Bind the User

Next, bind the `learner` user to the `node-reader` `ClusterRole`. You can also do this either with a YAML manifest or a direct command.

#### Option 1: Using a YAML Manifest

Create a YAML file named `node-reader-clusterrolebinding.yaml` with the following content:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-nodes-clusterwide
subjects:
- kind: User
  name: learner  # Name of the user being bound to the ClusterRole
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-reader  # Reference to the ClusterRole you created
  apiGroup: rbac.authorization.k8s.io
```

Apply the `ClusterRoleBinding`:

```bash
kubectl apply -f node-reader-clusterrolebinding.yaml
```

#### Option 2: Using a Direct `kubectl` Command

You can also create the `ClusterRoleBinding` using a single command:

```bash
kubectl create clusterrolebinding read-nodes-clusterwide \
  --clusterrole=node-reader \
  --user=learner
```

### 3. Verify User Permissions

Once the `ClusterRole` and `ClusterRoleBinding` have been created, switch to the `learner` context and test the following:

1. **List Nodes**:

   ```bash
   kubectl get nodes
   ```

   **Expected Outcome**: The `learner` user should be able to list all nodes in the cluster.

2. **Get Detailed Node Information**:

   ```bash
   kubectl get node <node-name> -o yaml
   ```

   **Expected Outcome**: The `learner` user should be able to retrieve detailed information about any node in the cluster.

3. **Create or Modify Nodes (Fail Test)**:

   Since nodes are created by the control plane, there’s no direct command to create them manually. However, any attempt to modify node objects should fail.

   **Expected Outcome**: The `learner` user will not be able to create or modify nodes, as the `ClusterRole` only grants read access.

### Summary

- **ClusterRole**: Grants read-only access to nodes (`list`, `get`, `watch`).
- **ClusterRoleBinding**: Binds the `learner` user to the `node-reader` `ClusterRole`.

