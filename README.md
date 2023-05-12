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

# What is PDB:

PDB stands for Pod Disruption Budget. It is a Kubernetes resource that allows you to define the minimum availability requirements for your application during disruptive events such as node maintenance, cluster upgrades, or resource constraints. PDB ensures that a certain number of Pods are available and prevents excessive downtime or unavailability of critical components.

# Here's an example of how to use a Pod Disruption Budget:

```yaml
apiVersion: policy/v1beta1
kind: PodDisruptionBudget
metadata:
  name: my-pdb
spec:
  selector:
    matchLabels:
      app: my-app
  maxUnavailable: 1
```

# In this example:

- The selector field selects the Pods to which the Pod Disruption Budget applies. In this case, it selects Pods with the label app: my-app.
- The maxUnavailable field defines the maximum number of Pods that can be unavailable at any given time. In this case, it allows at most 1 Pod to be unavailable.
- When a disruptive event occurs, such as a node being drained for maintenance or a cluster upgrade, Kubernetes checks the Pod Disruption Budget to ensure it doesn't violate the defined availability requirements. If the event would exceed the specified maxUnavailable value, the disruption is not allowed, and the operation is postponed until the necessary number of Pods are available.
- By setting appropriate values for maxUnavailable, you can control the availability of your application during planned or unplanned disruptions. For example, setting maxUnavailable: 1 ensures that at least one Pod remains available during disruptions, providing a certain level of redundancy and availability.
- It's important to note that the actual availability of your application also depends on factors such as the number of replicas, resource constraints, and the overall health of your cluster. PDBs are a tool to define and enforce availability policies, but they should be used in conjunction with other Kubernetes features and best practices to achieve the desired level of application resilience.

# How to schedule pod across the AZ and different type of k8s topologyKey.

In Kubernetes, topologyKey is a field used in various scheduling configurations to define the topology domain on which scheduling decisions are made. It represents a label or annotation key that identifies a specific topology domain, such as a node, zone, region, or other custom topology domains. The value of the topologyKey is used to group resources together based on the defined domain.

# Different Types of topologyKey for scheduling in Kubernetes:

## topologyKey: kubernetes.io/hostname:

This topologyKey represents the node's hostname.
It can be used to ensure Pods are scheduled onto specific nodes based on their hostname.

- Use this topology key when you want to schedule Pods based on specific node hostnames.
- It allows you to target Pods to run on specific nodes identified by their hostname.
- 
### Example use case:

Let's say you have nodes with different hardware capabilities, and you want to ensure that Pods with certain resource requirements are scheduled on nodes with specific hostnames that indicate their hardware capabilities. You can use topologyKey: kubernetes.io/hostname to match the Pod's scheduling requirements with the corresponding node hostname.

### Example syntax usage:

```yaml
affinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
      - matchExpressions:
          - key: kubernetes.io/hostname
            operator: In
            values:
              - node1
              - high-capacity-node
```

###  Here's an example of scheduling a Pod using the topologyKey: kubernetes.io/hostname:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
    - name: my-container
      image: nginx
  affinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - node1
```                
In this example:

- The Pod is scheduled to run on a node with the hostname node1.
- The affinity field specifies the scheduling requirements for the Pod.
- The requiredDuringSchedulingIgnoredDuringExecution policy ensures that the Pod is only scheduled on nodes that match the specified criteria.
- The nodeSelectorTerms field is used to define the node selection criteria.
- The matchExpressions field specifies the key-value pairs to match against the node labels.
- The key: kubernetes.io/hostname indicates that the Pod should be scheduled on a node with a matching hostname.
- The values field specifies the allowed values for the hostname. In this case, it is set to node1.
With this configuration, the Pod will only be scheduled on a node with the hostname node1. If there are no nodes with that hostname or if they are already fully utilized, the Pod will remain unscheduled until a suitable node becomes available.

You can modify the values field to specify different hostnames or add additional expressions to match against multiple hostnames or patterns. Remember to ensure that the specified hostnames correspond to the actual node hostnames in your cluster.

Note that other factors, such as resource availability, node constraints, and Pod affinity/anti-affinity rules, can also affect the scheduling decision in conjunction with the topologyKey: kubernetes.io/hostname configuration.


## topologyKey: topology.kubernetes.io/zone:

This topologyKey represents the availability zone (AZ) of a node. It can be used to distribute Pods across different availability zones.

- Use this topology key when you want to distribute Pods across different availability zones (AZs).
- It allows you to ensure high availability and fault tolerance by spreading Pods across different AZs within a cluster.

### Example use case:

In a multi-AZ Kubernetes cluster, you can use topologyKey: topology.kubernetes.io/zone to enforce anti-affinity rules and distribute Pods of the same application across different AZs. This ensures that if one AZ becomes unavailable, the application continues to run on nodes in other AZs.

### Example syntax usage:

```yaml
affinity:
  podAntiAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
            - key: app
              operator: In
              values:
                - my-app
        topologyKey: topology.kubernetes.io/zone
```

###  Here's an example of scheduling a Pod using the topologyKey: kubernetes.io/zone

To configure a Kubernetes application to ensure that replicas are distributed across availability zones (AZs), you can use the Pod anti-affinity feature along with the topologySpreadConstraints. This allows you to specify rules that influence the scheduling of Pods across different AZs. Here's an example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                topologyKey: topology.kubernetes.io/zone
                labelSelector:
                  matchLabels:
                    app: my-app
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: topology.kubernetes.io/zone
          whenUnsatisfiable: DoNotSchedule
          labelSelector:
            matchLabels:
              app: my-app
```              
In this example:

- The podAntiAffinity field specifies that Pods of the same application (app: my-app) should be anti-affinitized, meaning they should not be scheduled on the same node or within the same AZ.
- preferredDuringSchedulingIgnoredDuringExecution sets a preference for scheduling across different AZs.
- topologyKey: topology.kubernetes.io/zone indicates that AZ is the topology key used for anti-affinity.
- The topologySpreadConstraints field ensures that Pods are spread across different AZs.
- maxSkew: 1 specifies that there should be at most one more Pod in any AZ than the AZ with the fewest Pods.
- whenUnsatisfiable: DoNotSchedule ensures that if the constraint cannot be satisfied, the Pod will not be scheduled instead of being scheduled without adhering to the constraint.
- With this configuration, Kubernetes will attempt to distribute the Pods of the my-app deployment across different AZs. This improves availability and resilience by avoiding a single point of failure within an AZ.

Note that the effectiveness of distributing Pods across AZs depends on the availability of nodes within those AZs and the configuration of your underlying infrastructure. Additionally, ensure that your cluster has multiple nodes across different AZs for this configuration to take effect.

It's recommended to test and validate the distribution of Pods across AZs in your specific Kubernetes environment and consult the documentation for additional information on anti-affinity and AZ-aware scheduling in your Kubernetes distribution.


## topologyKey: kubernetes.io/hostname: & topologyKey: topology.kubernetes.io/zone:

By combining the use of topologyKey with kubernetes.io/hostname and topology.kubernetes.io/zone, you can fine-tune the scheduling of Pods based on specific hostnames and distribute them across different availability zones to achieve high availability and optimized resource allocation.

### Here's an example of using topologyKey with both kubernetes.io/hostname and topology.kubernetes.io/zone in a YAML manifest:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - my-app
              topologyKey: topology.kubernetes.io/zone
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
            - matchExpressions:
                - key: kubernetes.io/hostname
                  operator: In
                  values:
                    - high-capacity-node
      containers:
        - name: my-container
          image: nginx
```

In this example:

- The podAntiAffinity section with topologyKey: topology.kubernetes.io/zone ensures that Pods of the same application (my-app) are scheduled across different availability zones. This provides high availability and fault tolerance by distributing the Pods across multiple zones.
- The requiredDuringSchedulingIgnoredDuringExecution section with topologyKey: kubernetes.io/hostname specifies that the Pods should be scheduled on nodes with the hostname high-capacity-node. 
- This targets Pods to run on nodes with specific hostnames that indicate high-capacity hardware capabilities.

### With this configuration:

- The topologyKey: topology.kubernetes.io/zone ensures that the Pods of my-app are distributed across different availability zones, providing resilience in case of zone failures.
- The topologyKey: kubernetes.io/hostname ensures that the Pods are scheduled on nodes with the hostname high-capacity-node, which indicates nodes with higher resource capacity.

Note that in a real deployment, you would need to replace high-capacity-node with the actual hostnames and configure the appropriate labels and node selectors based on your cluster configuration.

## topologyKey: topology.kubernetes.io/region:

This topologyKey represents the region of a node.
It can be used to distribute Pods across different regions.

### Example syntax usage:

```yaml
topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/region
    whenUnsatisfiable: ScheduleAnyway
```    

## Custom Topology Keys:

You can also define your own custom topology keys based on your infrastructure requirements.
### Example syntax usage:

```yaml
affinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
      - matchExpressions:
          - key: my-custom-topology-key
            operator: In
            values:
              - value1
              - value2
```

It's important to note that the availability and behavior of these topology keys may vary depending on the Kubernetes distribution and underlying infrastructure. Consult the documentation and resources specific to your Kubernetes distribution for detailed information on supported topology keys and their usage in scheduling configurations.

