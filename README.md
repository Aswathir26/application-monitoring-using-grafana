# **Prometheus and Grafana**

Prometheus and Grafana becoming most common monitoring platform in microservices based devops infrastructure. Prometheus is a powerful time series metrics collection and alerting system. Grafana is a visualization tools which can be used with Prometheus.

Metrics collection of Prometheus follows the pull model. That means, Prometheus is responsible for getting metrics from the services that it monitors. This process introduced as scraping. Prometheus server scrapes the defined service endpoints, collect the metrics and store in local database. Then Grafana can visualize them in various graphs and charts. 

# **Expose metrics**

In order to Prometheus to scrape the metrics, each service need to expose their metrics(with label and value) via HTTP endpoint '/metrics'. For an example if I want to monitor a microservice with Prometheus I can collect the metrics from the service and expose them with HTTP endpoint. To do that I need to create HTTP server with /metrics endpoint inside the microservice. Prometheus comes with different client libraries to do that.

# **Exporters**

Promethus provides a way to monitory third party applications and services with Exporters. Exporters act as side-car to third party application/services. They collect data from third party applications/services and expose them with HTTP endpoint which Prometheus can scrape. There are various exporters available with Prometheus. The most common exporter is node exporter, which can be installed on every server to read system level metrics such as cpu, memory, file system etc. node exporter is used to collect data on all nodes.
 
To collect data from all the nodes within the Kubernetes cluster, you can deploy a DaemonSet. A DaemonSet allows for some (or all) pods to be scheduled and run on all nodes. The DaemonSet Controller adds a new pod to every new node that is added to the Kubernetes cluster. When a node is removed, the DaemonSet pod is garbage collected. This is why the Node Exporter should be deployed as a DaemonSet in Kubernetes.

# **Grafana-Loki**

Loki is a log aggregation system designed to store and query logs from all your applications and infrastructure.
Loki is a horizontally scalable, highly available, multi-tenant log aggregation system inspired by Prometheus. It is designed to be very cost effective and easy to operate. It does not index the contents of the logs, but rather a set of labels for each log stream.

# **Promtail**
Promtail is an agent which ships the contents of local logs to a private Grafana Loki instance or Grafana Cloud. It is usually deployed to every machine that has applications needed to be monitored.

It primarily:

- Discovers targets
- Attaches labels to log streams
- Pushes them to the Loki instance.

**Ingress**

In Kubernetes, an Ingress is an object that allows access to Kubernetes services from outside the Kubernetes cluster. You can configure access by creating a collection of rules that define which inbound connections reach which services.

An Ingress can be configured to give Services externally-reachable URLs, load balance traffic, terminate SSL/TLS, and offer name-based virtual hosting. Ingress lets you configure an HTTP load balancer for applications running on Kubernetes, represented by one or more Kubernetes internal Services.

**Ingress Controller**

An ingress controller is a piece of software that provides reverse proxy, configurable traffic routing, and TLS termination for Kubernetes services. Kubernetes ingress resources are used to configure the ingress rules and routes for individual Kubernetes services. When you use an ingress controller and ingress rules, a single IP address can be used to route traffic to multiple services in a Kubernetes cluster.

An Ingress controller is responsible for fulfilling the Ingress, usually with a load balancer, though it may also configure your edge router or additional frontends to help handle the traffic.


**Note: Here we are using unified alerting system**



Follow the given steps to monitor Applications, Kubernetes and Ingress controller

#STEPS
-----

**step 1:**

    helm repo add bitnami https://charts.bitnami.com/bitnami
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

    helm repo update

**step 2:**

    kubectl create namespace nets

**step 3:**

for logs monitoring

    cd loki
    helm install loki -f values.yaml bitnami/grafana-loki

**step 4:**

for metrics/kubernetes monitoring

    cd prometheus
    helm install logs -f values.yaml prometheus-community/kube-prometheus-stack

**step 5:**

for nginx ingress-controller monitoring (using prometheus)

    cd ingress
    kubectl apply -f keycloak_ingress.yaml
    helm install kclk-ingress -f values.yaml bitnami/nginx-ingress-controller -n nets

   
**step 6:**

    cd dashboard
    kubectl create configmap my-dashboard-application --from-file=Application_db.json
    kubectl create configmap my-dashboard-kubernetes --from-file=Kubernetes_db.json
    kubectl create configmap my-dashboard-ingresscontroller --from-file=Ingresscontroller_db.json
    

**step 7:**


    cd grafana
    kubectl apply -f datasource.yaml
    helm install grafana -f values.yaml bitnami/grafana

    1. Get the application URL by running these commands:
    echo "Browse to http://127.0.0.1:8080"
    kubectl port-forward svc/grafana 8080:3000 

    2. Get the admin credentials:

    echo "User: admin"
    echo "Password: $(kubectl get secret grafana-admin --namespace default -o jsonpath="{.data.GF_SECURITY_ADMIN_PASSWORD}" | base64 -d)"
---------------------------------------------------------------------------------------------------

