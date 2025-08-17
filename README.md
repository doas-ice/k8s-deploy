# Kubernetes Deployment Strategies with Argo CD  


## Project Overview  

 
This project demonstrates containerizing a Node.js application, pushing it to Docker Hub, and deploying it on a Kubernetes cluster using three deployment strategies — Blue-Green, Rolling Update, and Canary — managed by Argo CD.

Ingress is used to expose the application outside the cluster with a single entry point, enabling browser access without manual kubectl port-forward.

---

# 🔗 Links
```
    GitHub Repo: 🔗 k8s-deploy
    Docker Hub Image(s): 🔗 aureri/node-deployment-app
    Dockerfile: Dockerfile
    Kubernetes Manifests: k8s/
    ArgoCD Definitions: argocd/
```

# Repository Structure
```
├── Dockerfile
├── blue-green/
│   ├── deployment-blue.yaml
│   ├── deployment-green.yaml
│   ├── service.yaml
│   └── ingress.yaml
├── rolling/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
├── canary/
│   ├── deployment-canary.yml
│   ├── deployment-stable.yaml
│   ├── ingress-canary.yaml
│   ├── ingress-main.yaml
│   ├── service-canary.yaml
│   └── service-stable.yaml
└── argocd/
    ├── app-blue-green.yaml
    ├── app-rolling.yaml
    └── app-canary.yaml
```
# Screenshots

    Argo CD dashboard showing all three strategies
    Running pods for each strategy (kubectl get pods)
    Application in browser before and after update

# Assignment Questions & Answers

## Q1. Explain the difference between a Kubernetes Service, an Ingress controller (e.g., NGINX Ingress), and a cloud provider’s load balancer (e.g., AWS ELB). Why do Kubernetes clusters often use multiple layers of load balancing instead of just one?

    Kubernetes Service: A Service is an abstraction that defines a logical set of Pods and a policy to access them. It provides internal load balancing within the cluster and a stable IP/hostname for the Pods. For example, a Service ensures that if Pods are recreated or scaled up/down, other components can still reach the application reliably.

    Ingress Controller (e.g., NGINX Ingress): An Ingress Controller manages external HTTP(S) traffic into the cluster. It allows you to define rules based on hostnames or paths to route traffic to specific Services. Unlike a Service, which only balances traffic inside the cluster, Ingress gives you a single entry point for multiple services, supports SSL/TLS termination, and can handle path-based routing.

    Cloud Load Balancer (e.g., AWS ELB/ALB): A cloud LB is an external load balancer managed by your cloud provider. It routes internet traffic to your cluster nodes or services. Unlike Kubernetes Services or Ingress, it exists outside the cluster and often has features like global distribution, health checks, and autoscaling.

Why multiple layers? Kubernetes often uses a stacked approach: Cloud LB → Ingress → Service → Pod. Each layer has a purpose:

    Cloud LB handles external traffic and scales automatically.
    Ingress provides fine-grained HTTP routing and TLS termination.
    Service balances traffic internally across Pods.

This separation allows flexibility, scalability, security, and observability, instead of relying on a single monolithic load balancer.


## Q2. In a microservices architecture running on Kubernetes, why might you choose to use an Ingress controller like NGINX instead of exposing every Service with a LoadBalancer type service? Discuss benefits and potential drawbacks.

In a microservices architecture, exposing every Service as a LoadBalancer can quickly become expensive and hard to manage, because most cloud providers create a dedicated external LB per Service.

Benefits of using an Ingress controller:

    Single entry point for all services → easier DNS management.
    Cost-efficient → fewer cloud load balancers needed.
    Advanced routing → can route based on path, hostname, or even headers.
    TLS termination → SSL certificates can be centralized.

Drawbacks / considerations:

    Adds an extra layer of complexity; you need to manage and maintain the Ingress Controller.
    Some features, like advanced traffic splitting by percentage, require additional tools (NGINX annotations, Service Mesh, or Argo Rollouts).

Overall, Ingress is preferred when you have many services and want centralized traffic control.


## Q3. Describe how Blue-Green and Canary deployment strategies influence traffic routing and load balancing within Kubernetes. Why can’t you simply rely on Kubernetes Services for sophisticated traffic splitting required by Canary deployments?

    Blue-Green Deployment: You maintain two identical environments — Blue (current) and Green (new). All traffic is routed to Blue initially. When Green is ready, you switch 100% of traffic to Green by updating the Service or Ingress.
        Pros: Instant rollback is simple; traffic is fully isolated.
        Cons: Requires double the resources while both environments are running.

    Canary Deployment: You release a new version (canary) to a small subset of traffic (e.g., 10%) while the majority continues hitting the stable version. Gradually, the canary traffic is increased until 100%.
        Pros: Safer rollout; you can detect issues early.
        Cons: Requires precise traffic routing and observability.

Why Services alone are not enough for Canary: Kubernetes Services do evenly distribute traffic across all pods. They cannot split traffic by percentage or target only a subset of pods. Canary releases need either Ingress annotations or a Service Mesh to direct specific percentages of traffic to the new version.


## Q4. What are the advantages and disadvantages of using a cloud provider’s load balancer (such as AWS ELB/ALB) directly in front of Kubernetes clusters compared to using an Ingress controller or Service of type LoadBalancer?

    Cloud Load Balancer (AWS ELB/ALB)
        Pros: Managed, reliable, globally distributed, integrates with cloud networking.
        Cons: Expensive if you need one per Service; less flexible routing for internal HTTP paths.

    Ingress Controller
        Pros: Centralized entry, path/host routing, SSL termination, cost-effective.
        Cons: Needs to be installed and maintained; some advanced features require extra setup.

    Service of type LoadBalancer
        Pros: Simple, cloud-native, easy to expose a service externally.
        Cons: Creates one LB per service → can get costly; limited routing rules.

Summary: Use cloud LB if you need external high availability; Ingress if you want HTTP routing and path-based traffic management; Service LoadBalancer is simple but can be expensive at scale.


## Q5. In a Kubernetes cluster, explain the role of the Service mesh (e.g., Istio) in managing traffic compared to traditional Load Balancers and Ingress controllers. Why might a Service mesh be preferred for advanced deployment strategies such as Canary rollouts?

A Service Mesh like Istio manages internal pod-to-pod traffic and provides fine-grained control over requests.

    Traditional Load Balancers and Ingress only operate at service boundaries.

    Service Mesh can:
        Split traffic by percentage, headers, or cookies.
        Handle retries, timeouts, and circuit breaking.
        Provide observability with metrics, logs, and tracing.

Why it’s preferred for advanced deployments like Canary:

    Canary rollouts often require precise traffic percentages and gradual shifting.
    Istio or other service meshes let you incrementally route traffic without creating multiple Ingress resources.
    You can also integrate with monitoring to automate rollback if errors spike.

In short, Service Mesh gives you control, observability, and safety that standard LoadBalancers or Ingress can’t easily provide.
