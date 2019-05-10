# Helm Chart for Splunk

This chart is based on the article [Deploy Splunk Enterprise on Kubernetes: Splunk Connect for Kubernetes and Splunk Insights for Containers (BETA) - Part 1][splunk-k8s]

## Web Service for Defaults

Before the Splunk chart can be deployed we need to create a web service that will serve up the default settings for Splunk including the licence file. This is done by configuring a simple nginx container.

First the Splunk default settings need to be created. This can be done using the splunk container image.

```shell
mkdir defaults
docker run --rm splunk/splunk create-defaults > ./defaults/default.yml
```

Copy your Splunk licence file `mySplunkLicense.lic` into this directory, then deploy this to Kubernetes as a [configmap][configmap]. This file will then be mounted into the nginx application.

Note: all the Splunk deployments need to be in a Kubernetes namespace 'splunk'

```shell
kubectl create namespace splunk

kubectl -n splunk create configmap nginx-data-www --from-file=./defaults
```

Now you can go ahead and deploy the defaults web service, using helm.

The default admin user password for splunk is configured via a Kubernetes secret. The secret is also created as part of the Helm chart and needs to be passed in when you install the chart.

```shell
helm install splunkdefaults --namespace splunk --set splunk_password='your password'
```

If you want to verify this is working you can use the `kubectl proxy` command and view the following link in a browser

 * http://localhost:8001/api/v1/namespaces/splunk/services/http:splunk-defaults:nginx-web/proxy/

Now deploy Splunk itself.

```shell
helm install splunkchart --namespace splunk
```

The Splunk UI doesn't work with the proxy URLs, but you can set up port forwarding to test.

First get the name of the pod for the master

```shell
kubectl get pods -n splunk
NAME                               READY   STATUS    RESTARTS   AGE
indexer-0                          1/1     Running   0          7m20s
indexer-1                          1/1     Running   0          5m27s
indexer-2                          1/1     Running   0          4m37s
master-7d9777994d-8xcrd            1/1     Running   0          7m20s
search-7f888c8c4c-4wzgg            1/1     Running   0          7m20s
splunk-defaults-58d8cdff6f-p5qc2   1/1     Running   0          8m
```

Then run 

```shell
kubectl port-forward -n splunk master-7d9777994d-8xcrd 8000:8000
```

 * http://localhost:8000/

If you have direct conectivity to the virtual network, for example from a VM or via VPN, then you can also access the UI via the internal loadbalancer endpoint. To find the IP address, use:

```shell
kubectl get service webui -n splunk
```


[splunk-k8s]: https://www.splunk.com/blog/2018/12/17/deploy-splunk-enterprise-on-kubernetes-splunk-connect-for-kubernetes-and-splunk-insights-for-containers-beta-part-1.html "Deploy Splunk Enterprise on Kubernetes"

[configmap]: https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/