# Shinobi on Kubernetes

This project provides a set of Kubernetes manifests to deploy [Shinobi](https://shinobi.video/), an open-source CCTV and NVR (Network Video Recorder) solution.

## Project Structure

The Kubernetes manifests are located in the `k8s/` directory:

- `shinobi-namespace.yaml`: Creates a dedicated namespace `shinobi-ns` for all our resources.
- `shinobi-pv.yaml`: Defines two `PersistentVolume`s using `hostPath`. These are used to store the MariaDB database and Shinobi's configuration/video files.
- `shinobi-pvc.yaml`: `PersistentVolumeClaim`s that request storage from the `PersistentVolume`s.
- `shinobi-deployment.yaml`: Deploys the Shinobi application. It uses the official development image and mounts the persistent volumes for data storage.
- `shinobi-service.yaml`: Exposes the Shinobi web UI and FTP service using a `NodePort` service.

## Prerequisites

- A running Kubernetes cluster.
- `kubectl` installed and configured to communicate with your cluster.
- Host directories for persistent storage.

## Deployment

### 1. Prepare Host Paths

This deployment uses `hostPath` for persistent storage, which means it relies on specific directories existing on the Kubernetes node where the Shinobi pod is scheduled.

You **must** create these directories on the node(s) before deploying:

```bash
mkdir -p /home/jahangir/ShinobiDatabase
mkdir -p /home/jahangir/Shinobi
```

> **Note:** For a production setup, it is highly recommended to replace the `hostPath` volumes with a more robust storage solution like a network-attached storage (NFS) provisioner or a cloud provider's block storage.

### 2. Apply the Manifests

Deploy all the resources to your cluster by applying the manifests in the `k8s` directory.

First, apply the namespace:
```bash
kubectl apply -f k8s/shinobi-namespace.yaml
```

Then, apply the rest of the resources:
```bash
kubectl apply -f k8s/ --namespace=shinobi-ns
```

This will create the PersistentVolumes, PersistentVolumeClaims, Deployment, and Service in the `shinobi-ns` namespace.

### 3. Accessing Shinobi

The Shinobi web interface is exposed via a `NodePort`. To access it, you need the IP address of one of your Kubernetes nodes and the port specified in the service.

1.  Find the `NodePort` for the `http` service:

    ```bash
    kubectl get svc shinobi-service -n shinobi-ns
    ```

    The output will show the port mappings. By default, it is `30080`.

2.  Get the IP address of your Kubernetes nodes:

    ```bash
    kubectl get nodes -o wide
    ```

3.  Open your web browser and navigate to:

    `http://<NODE_IP_ADDRESS>:30080`

## Cleanup

To remove all the resources created by this deployment, run the following command:

```bash
kubectl delete -f k8s/
```