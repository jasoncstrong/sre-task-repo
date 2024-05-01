
# Week 1 Project
![](https://uplimit.com/ugc-assets/course/course_cls1f66i100or12c4029h4wrx/assets/1711750513648-4x6ljmxwklb3v5jhywsy4nmf2b3kvv/up.png)

Welcome to UpCommerce! You've recently joined the ranks of a fast-growing, early-stage startup as the first hire with a focus on Site Reliability Engineering (SRE). Over the next four weeks, you'll apply your skills to strengthen the company's infrastructure, reliability, and overall performance as it continues its growth.

## Introduction

In this project, you'll be tasked with deploying UpCommerce's website as a service on Kubernetes. You'll define alert rules based on key performance indicators (KPIs) like CPU utilisation, memory usage, and request rates, allowing you to detect and respond to potential issues before they affect the user experience. You'll create and deploy these rules using Prometheus. Prometheus is an open-source monitoring and alerting system designed for containerized environments, such as Kubernetes. It collects and stores metrics from various sources, including applications, infrastructure components, and custom exporters, allowing you to monitor the performance and health of your systems in real-time. Prometheus uses a powerful query language called PromQL to analyze and visualize the collected metrics, enabling you to gain valuable insights into the behavior of your applications and infrastructure.

By the end of this project, you'll have a comprehensive view of the UpCommerce website, enabling you to identify trends, patterns, and potential issues that may affect the user experience or the overall stability of the system. You'll also have gained hands-on experience in managing a production-grade e-commerce platform and the skills necessary to maintain its reliability and performance.

## Getting your development environment ready

For this project, you'll be using GitHub Codespaces to deploy UpCommerce's service infrastructure because GitHub Codespaces has all the tools we require for this project (Minikube, Docker, Helm, Git, etc). If this is your first encounter with GitHub Codespaces, you can watch the video below[,](https://uplimit.com/course/sre-fundamentals-with-google/v2/assignment/assignment_cltz11je4003r12868smdbk4i) and you'll get the scoop on how to create and use a GitHub codespace.

Here's a video that shows how [upcommerce.com](http://upcommerce.com/) is served on Kubernetes, using Minikube in GitHub Codespaces:

Here are the steps to take as you begin your project:

### Steps

1.  Create a fork of the [week's repository](https://github.com/onyekaugochukwu/sre-task-repo).

2.  When you have created a fork of the week's repository, start a codespace on the `main` branch (unless you created another branch after you created your fork of the repository).

3.  Run the command below in your Codespace's terminal to create a single-node, Kubernetes cluster using Minikube:

    minikube start

4.  Once your Minikube cluster is running, enter the command below:

    kubectl create namespace sre

This creates a namespace in your Kubernetes cluster named `sre`. It is within this namespace that you'll do all the tasks required for this project.

 -  **Task 1:** Define your Prometheus configuration in the `prometheus.yml` file before you run the command in Step 6 below. Define the alert rules and triggers that should fire off the alerts. You need to add the following alerts to the `prometheus.yml` file by changing the values of all the fields marked with "#TODO" in the `prometheus.yml` file:

**(a) Node Down Alert Group**

 -   This alert should be triggered under the following scenarios:
	- When a Kubernetes node is unreachable for more than 5 minutes.
 -   The alert rule should include:
	- Alert Name: InstanceDown
	- Expression: **`up{job="kubernetes-nodes"} == 0`**
	- Trigger Duration: 2 minutes
	- Labels:
		- severity: page
	- Annotations:
		- host: "{{ $labels.kubernetes_io_hostname }}"
		- summary: "Instance down"
		- description: "Node {{ $labels.kubernetes_io_hostname }} has been down for more than 5 minutes."

**(b) Low Memory Alert**

 - This alert should be triggered when a Kubernetes node's available
   memory falls below 85%. 
- The alert rule should include:
	- Alert Name: LowMemory
	-   Expression: `(node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100 < 85`
	-   Trigger Duration: 2 minutes
	-   Labels:
		-   severity: warning
	-   Annotations:
		-   host: "{{ $labels.kubernetes_node }}"
		-   summary: "{{ $labels.kubernetes_node }} Host is low on memory. Only {{ $value }}% left"
		-   description: "{{ $labels.kubernetes_node }} node is low on memory. Only {{ $value }}% left"

**(c) Kube Persistent Volume Errors Alert**
-   This alert should be triggered if any persistent volume has a status of "Failed" or "Pending".
-   The alert rule should include:
	-   Alert Name: KubePersistentVolumeErrors
	-   Expression: `kube_persistentvolume_status_phase{job="kubernetes-service-endpoints",phase=~"Failed|Pending"} > 0`
	-   Trigger Duration: 2 minutes
	-   Labels:
		-   severity: critical
	- Annotations:
		-   description: The persistent volume {{ $labels.persistentvolume }} has status {{ $labels.phase }}.
		-   summary: PersistentVolume is having issues with provisioning.

**(d) Kube Pod Crash Looping Alert**
-   This alert should be triggered if any Kubernetes pod is restarting more than once every 5 minutes.
-   The alert rule should include:
	-   Alert Name: KubePodCrashLooping
	-   Expression: `rate(kube_pod_container_status_restarts_total{job="kubernetes-service-endpoints",namespace=~".*"}[5m]) * 𝟼𝟶 * 𝟻 > 𝟶`
	-   Trigger Duration: 2 minutes
	-   Labels:
		-   severity: warning
	-   Annotations:
		-   description: Pod {{ $labels.namespace }}/{{ $labels.pod }} ({{ $labels.container }}) is restarting {{ printf "%.2f" $value }} times / 5 minutes.
	-   summary: Pod is crash looping.

**(e) Kube Pod Not Ready Alert**
-   This alert should be triggered if any Kubernetes pod remains in a non-ready state for longer than 2 minutes.
-   The alert rule should include:
	-   Alert Name: KubePodNotReady
	-   Expression: `sum by(namespace, pod) (max by(namespace, pod) (kube_pod_status_phase{job="kubernetes-service-endpoints",namespace=~".*",phase=~"Pending|Unknown"}) * on(namespace, pod) group_left(owner_kind) topk by(namespace, pod) (1, max by(namespace, pod, owner_kind) (kube_pod_owner{owner_kind!="Job"}))) > 0`
	-   Trigger Duration: 2 minutes
	-   Labels:
		-   severity: warning
	-   Annotations:
		-   description: `Pod {{ $labels.namespace }}/{{ $labels.pod }} has been in a non-ready state for longer than 5 minutes`.
		-   summary: `Pod has been in a non-ready state for more than 2 minutes."`

6.  Once you have declared all the alerts in the `prometheus.yml` file, run the command below:

    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    helm repo update
    helm install prometheus prometheus-community/prometheus \
      -f prometheus.yml \
      --namespace sre \

Here's what the commands above are doing:

-   **`helm repo add prometheus-community`** [**`https://prometheus-community.github.io/helm-charts`**](https://prometheus-community.github.io/helm-charts): This command adds a Helm repository named `prometheus-community` and sets its URL to [`https://prometheus-community.github.io/helm-charts`](https://prometheus-community.github.io/helm-charts). Helm repositories are used to store and distribute Helm charts, which are packages of pre-configured Kubernetes resources.med

-   **`helm repo update`**: This command updates the local cache of Helm charts from all configured Helm repositories. It ensures that you have the latest information about available charts and their versions.

-   **`helm install prometheus prometheus-community/prometheus -f prometheus.yml --namespace sre`**: This command installs the Prometheus chart from the **`prometheus-community`** repository into the Kubernetes cluster. Here's what each part of the command does:

-   **`helm install prometheus`**: This specifies the name of the release. In this case, the release will be named `prometheus`.

-   **`prometheus-community/prometheus`**: This specifies the chart to install, which is `prometheus` from the `prometheus-community` repository.

-   **`-f prometheus.yml`**: This flag specifies a YAML file (`prometheus.yml`) containing custom configuration values for the Prometheus chart. This file will override default configuration values defined in the chart.

-   **`--namespace sre`**: This flag specifies the namespace (`sre`) in which to install the Prometheus resources.

7.  Run the code below:

    helm repo add grafana https://grafana.github.io/helm-charts
    helm repo update
    helm install grafana grafana/grafana \
     --namespace sre \
     --set adminPassword="admin"

Here's what the above code does:

-   **`helm repo add grafana`** [**`https://grafana.github.io/helm-charts`**](https://grafana.github.io/helm-charts): This command adds a Helm repository named `grafana` and sets its URL to [`https://grafana.github.io/helm-charts`](https://grafana.github.io/helm-charts). Helm repositories are used to store and distribute Helm charts, which are packages of pre-configured Kubernetes resources for Grafana in this case.

-   **`helm repo update`**: This command updates the local cache of Helm charts from all configured Helm repositories. It ensures that you have the latest information about available charts and their versions.

-   **`helm install grafana grafana/grafana`**: This command installs the Grafana chart from the `grafana` repository into the Kubernetes cluster. Here's what each part of the command does:

-   **`helm install grafana`**: This specifies the name of the release. In this case, the release will be named `grafana`.

-   **`grafana/grafana`**: This specifies the chart to install, which is `grafana` from the `grafana` repository.

-   **`--namespace sre`**: This flag specifies the namespace (`sre`) in which to install the Grafana resources. Namespaces provide a way to scope and isolate resources within a Kubernetes cluster.

-   **`--set adminPassword="admin"`**: This flag sets a custom configuration value for the Grafana chart. In this case, it sets the administrator password to `"admin"`. This password can be used to log in to the Grafana web interface.

8.  Run the code below to create a deployment and service

    kubectl apply -f deployment.yml -n sre
    kubectl apply -f service.yml -n sre

The code above performs the following functions:

-   **`kubectl apply -f deployment.yml -n sre`**:

-   **`kubectl apply`**: This command is used to apply a configuration to a Kubernetes cluster. It creates or updates resources based on the configuration provided.

-   **`-f deployment.yml`**: This flag specifies the file containing the configuration to apply. In this case, it's `deployment.yml`, which contains the definition for a Kubernetes Deployment resource.

-   **`-n sre`**: This flag specifies the namespace (`sre`) in which to apply the configuration. The configuration specified in `deployment.yml` will be applied to the `sre` namespace.

-   **`kubectl apply -f service.yml -n sre`**:

-   **`kubectl apply`**: Similar to the previous command, this command applies a configuration to a Kubernetes cluster.

-   **`-f service.yml`**: This flag specifies the file containing the configuration to apply. In this case, it's `service.yml`, which contains the definition for a Kubernetes Service resource.

-   **`-n sre`**: This flag specifies the namespace (`sre`) in which to apply the configuration. Again, the configuration specified in `service.yml` will be applied to the `sre` namespace.

9.  Create a [**split terminal**](https://uplimit.com/course/sre-fundamentals-with-google/v2/assignment/assignment_cltz11je4003r12868smdbk4i#corise_clu08tvoy002q356qzvn3mbxm) and Run the code below to ensure that UpCommerce's website is up

    minikube service upcommerce-service -n sre

This command creates a Minikube service and forwards it to a port where you can access it. You can try transacting on UpCommerce's site which is forwarded to this port using `admin` as both the username and password.

10.  Create another split terminal and run the following command:

    export POD_NAME=$(kubectl get pods --namespace sre -l "app.kubernetes.io/name=alertmanager,app.kubernetes.io/instance=prometheus" -o jsonpath="{.items[0].metadata.name}")
    kubectl --namespace sre port-forward $POD_NAME 9093

The above command allows you to forward your Prometheus AlertManager to a port within your codespace (usually port 9093)

11.  Create another split terminal and enter the command:

    export POD_NAME=$(kubectl get pods --namespace sre -l "app.kubernetes.io/name=prometheus,app.kubernetes.io/instance=prometheus" -o jsonpath="{.items[0].metadata.name}")
    kubectl --namespace sre port-forward $POD_NAME 9090

The above command allows you to forward your Prometheus server to a port within your codespace (usually port 9090).


### TEST YOUR K8S DEPLOYMENT

> To ensure your Kubernetes deployment is correct and working as
> expected, run the command:  

    minikube service upcommerce-service -n sre --url

> 
> Copy the IP address generated by Kubernetes for your
> UpCommerce website. Then enter the following command:
> 

    kubectl port-forward service/upcommerce-service -n sre 30768:5000

> 
> Port 5000 is the port in your `service.yml` file. Click on the Ports
> Tab on the Terminal Console. Next, Ctrl + click on the URL for the
> service port (30768)
> ![](https://uplimit.com/ugc-assets/course/course_cls1f66i100or12c4029h4wrx/assets/1711355365968-vkjjgsl3l9g2sdg065mjn9fqgd13f1/serviceportforwarding.jpg)
> 
> When the homepage of UpCommerce's website is displayed, login with the
> following credentials:
> 
> **username:** admin
> **password:** admin

## (Optional) Additional Tasks


### 1. Create an email alert

Your task is to write the following Prometheus alert manager configuration in a YAML file named `email-alert.yml`

**Instructions:**

1.  Set the `resolve_timeout` to 1 minute.

2.  Configure a receiver named `'gmail-notifications'` to send email notifications to [`example@gmail.com`](mailto:example@gmail.com).

-   Update `from`, `auth_username`, `auth_identity` with your Gmail email address.

-   Update `auth_password` with your Gmail app password.

-   Ensure `send_resolved` is set to `true`.

-   Customize the email headers and text to include alert details.

3.  Configure the routing rules:

-   `group_wait`: 10 seconds

-   `group_interval`: 2 minutes

-   `receiver`: `'gmail-notifications'`

-   `repeat_interval`: 2 minutes


Feel free to use Gmail or your favourite email account to test out the alert.

### 2. Create a Slack alert

Your task here is to write a Prometheus alert manager configuration for Slack in a YAML file named `slack-alert.yml` based on the provided specifications:

**Instructions:**

1.  Set the `resolve_timeout` to 1 minute.

2.  Configure the `slack_api_url` with the appropriate Slack webhook URL.

3.  Define a receiver named `'slack-notifications'` to send notifications to the `#upcommerce-devs` Slack channel (you'll need to create this in your Slack just for this project)

4.  Ensure `send_resolved` is set to `true` to send resolved alerts.

5.  Configure the routing rules:

-   `group_interval`: 5 minutes
-   `group_wait`: 10 seconds
-   `receiver`: `'slack-notifications'`
-   `repeat_interval`: 1 hour


To successfully execute this task, you'll need a Slack API URL. [Here's a guide](https://www.svix.com/resources/guides/how-to-get-slack-webhook-url/) that shows how to generate a Slack Webhook URL.

### SAFEGUARD YOUR SLACK AND EMAIL SECRETS

> For all of the above optional tasks, please use a secrets manager like
> GitHub Secrets, Kubernetes Secrets or any other secrets management
> system you prefer to prevent your Slack or email secrets from being
> pushed to your GitHub repo.

### How to create a split terminal
 **Steps**

1.  Click on **Terminal Selector** drop-down to display the various terminal options available in your codespace

2.  Select **Split Terminal.**

3.  Select **bash** to open another bash terminal to the side of your current terminal.


![](https://uplimit.com/ugc-assets/course/course_cls1f66i100or12c4029h4wrx/assets/1710965944370-2qp5kcpwr12rhx6bwptcdc3nj5pjdh/splitterminal.jpg)
