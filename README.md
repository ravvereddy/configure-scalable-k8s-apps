# configure-scalable-k8s-apps
Best practices for ensuring your app is HA, FT.

- Deploying a highly available, fault-tolerant, and scalable application in Kubernetes requires a combination of best practices, architecture considerations, and appropriate configuration settings. Here's an example of a Kubernetes application manifest that incorporates these principles:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: your-app-image:latest
          ports:
            - containerPort: 8080
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 15
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 10
            periodSeconds: 5
          resources:
            limits:
              cpu: 1
              memory: 1Gi
            requests:
              cpu: 0.5
              memory: 512Mi
```

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
    - port: 80
      targetPort: 8080
  type: LoadBalancer
```

- Explanation of key components and best practices used in the manifest:

## Deployment:

 - replicas: 3: Specifies that the application should have three instances for high availability and fault tolerance.
 - strategy: Configures rolling updates for zero-downtime deployments.
 - selector: Matches the labels to ensure all replicas are part of the deployment.

## Pod template:

 - livenessProbe: Defines an HTTP endpoint to periodically check if the application is alive.
 - readinessProbe: Defines an HTTP endpoint to check if the application is ready to receive traffic.
 - resources: Sets CPU and memory requests and limits to ensure resource allocation and container resource usage.

## Service:

 - selector: Selects the Pods belonging to the application deployment.
 - type: LoadBalancer: Exposes the application externally using a load balancer.

## Additional considerations and best practices:

 - Use appropriate resource requests and limits based on your application's requirements and available cluster resources.
 - Implement health checks (livenessProbe and readinessProbe) to ensure the application's availability and readiness.
 - Leverage a Deployment with multiple replicas for high availability and fault tolerance.
 - Consider using a StatefulSet for applications that require stable network identities and ordered scaling.
 - Use a LoadBalancer service type or an Ingress controller to expose the application externally.
 - Implement proper monitoring, logging, and observability solutions to ensure visibility into the application's health and performance.
 - Consider using Pod anti-affinity rules to distribute application replicas across different nodes or availability zones to improve fault tolerance.

Please note that the example provided is a basic starting point, and you may need to customize it based on your specific application requirements and infrastructure setup.
