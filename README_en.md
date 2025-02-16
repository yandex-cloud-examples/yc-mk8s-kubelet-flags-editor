# `DaemonSet` to add `kubelet` flags

## Description 
What this `DaemonSet` will do: 

1. Use a Bash script to continuously monitor nodes for the required flags.
2. Copy the flags from the `ConfigMap` if they are missing.
3. Restart `kubelet`.

`DaemonSet` supports nodes running Docker runtime. 

## General way to run it:

1. Create a dedicated namespace for `DaemonSet`, so that it can run isolated:

```
kubectl apply -f kubelet-flag-editor-ns.yaml
```

1. Create a simple `ConfigMap`, with flags inside (mind the quotation marks):

```
kubectl apply -f kubelet-flag-editor-configmap --namespace="kubelet-flag-editor"
```

1. Now, let's move on to creating the `DaemonSet` itself:

```
kubectl apply -f  kubelet-flag-editor-ds.yaml
```

From now on, you can monitor the DaemonSet state. Where a flag is updated, the Pod will restart the `kubelet` process, and the new flags will be applied to the latter.


### Updating the flags

Use the following command to add additional flags when you need them: 

```kubectl edit configmap kubelet-flag-editor-configmap -o yaml -n kubelet-flag-editor```

However, it is worth noting that the flag check script just checks whether the flags are there rather than their values. Thus, if you update the values, the script may fail to recognize these changes as updates, ignoring them. Also, setting incorrect values (e.g., if one confuses `gc-image-high-threshold` with `gc-image-low-threshold` in the `ConfigMap` above) will impede the `kubelet` process from restarting, and the node status will switch to `not ready`. 
