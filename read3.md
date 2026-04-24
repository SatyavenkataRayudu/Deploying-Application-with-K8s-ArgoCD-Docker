# Kubernetes Notes: deployment.yaml and service.yaml

## deployment.yaml

This file describes a Kubernetes Deployment for the `demo-app` service.

Key sections:

- `apiVersion: apps/v1`
  - Uses the stable apps API for Deployments.

- `kind: Deployment`
  - Defines a Deployment controller that manages pod replicas.

- `metadata.name: demo-app`
  - The name of the Deployment object.

- `spec.replicas: 2`
  - The Deployment should keep 2 running pod replicas.

- `spec.selector.matchLabels`
  - Selects pods with `app: demo-app`.
  - This label must match labels under `template.metadata.labels`.

- `spec.template`
  - Defines the pod template used to create pods.

  - `metadata.labels.app: demo-app`
    - Labels the pods so the Deployment, Service, and any selector can find them.

  - `spec.containers`
    - The actual application container.

    - `name: demo-app`
      - Container name inside the pod.

    - `image: <YOUR_DOCKERHUB_USERNAME>/demo-app:latest`
      - Docker image used by the container.
      - Replace `<YOUR_DOCKERHUB_USERNAME>` with your Docker Hub username.

    - `ports.containerPort: 8080`
      - Exposes port 8080 inside the container.

    - `livenessProbe`
      - Checks whether the app is alive.
      - Uses HTTP GET on `/actuator/health` port `8080`.
      - If the probe fails, Kubernetes restarts the container.
      - `initialDelaySeconds: 30` waits 30 seconds before the first check.
      - `periodSeconds: 10` checks every 10 seconds.

    - `readinessProbe`
      - Checks whether the app is ready to receive traffic.
      - Uses the same health endpoint and port.
      - `initialDelaySeconds: 20` waits 20 seconds before the first ready check.
      - `periodSeconds: 5` checks every 5 seconds.
      - If the pod is not ready, the Service will not send traffic to it.

    - `resources`
      - `requests` tell Kubernetes the minimum resources the container needs.
        - `memory: 256Mi`
        - `cpu: 250m`
      - `limits` cap the maximum resources the container can use.
        - `memory: 512Mi`
        - `cpu: 500m`

## service.yaml

This file defines the Service that exposes the `demo-app` pods.

Key sections:

- `apiVersion: v1`
  - Uses the core API group for Services.

- `kind: Service`
  - Declares a Service resource.

- `metadata.name: demo-app-service`
  - The Service name used inside the cluster.

- `spec.type: LoadBalancer`
  - Exposes the Service externally through a cloud provider load balancer.
  - In managed Kubernetes, this will request a public IP.
  - If you run on a local cluster like Minikube, this may behave differently.

- `spec.selector.app: demo-app`
  - Matches the pods created by the Deployment.
  - The Service sends traffic to pods with this label.

- `spec.ports`
  - Defines the port mapping.

  - `protocol: TCP`
    - Uses TCP for traffic.

  - `port: 80`
    - The Service port exposed to users and other services.

  - `targetPort: 8080`
    - The port on the pods where the application listens.

## How they work together

- The Deployment creates and manages pods running `demo-app`.
- The Service finds those pods with the label `app: demo-app`.
- The Service exposes the app on port `80` and forwards traffic to pod port `8080`.
- Health probes make sure only healthy pods receive traffic.
- `replicas: 2` means the app can handle some failure and provide availability.

## Practical note

- Replace the placeholder image with a real Docker image before deploying.
- Ensure the application serves `/actuator/health` if using these probes.
- If you are not on a cloud provider, use a different Service type such as `NodePort` for local testing.
