# istio-bookinfo-tracing

This repo contains a modified version of the [bookinfo sample application](https://github.com/istio/istio/tree/master/samples/bookinfo).

The docker images containing traces are located at:

https://hub.docker.com/u/burnsie7

## Install Istio

Follow instructions as usual until you deploy the sample application.

https://istio.io/v1.9/docs/setup/getting-started/#download

```
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.9.2 TARGET_ARCH=x86_64 sh -
```

Add [settings and env vars](https://docs.datadoghq.com/tracing/setup_overview/proxy_setup/?tab=istio#istio-configuration-and-installation) to demo.yaml or copy the one from this repo:

```
cp ./demo-tracing.yaml ./istio-1.9.2/manifests/profiles/demo.yaml
```

Modify bookinfo.yaml to include datadog trace agent port info and updated images.

```
cp ./bookinfo-tracing.yaml ./istio-1.9.2/samples/bookinfo/platform/kube/bookinfo.yaml
```

Continue installation

```
cd istio-1.9.2
export PATH=$PWD/bin:$PATH
istioctl install --set profile=demo -y
kubectl label namespace default istio-injection=enabled
kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
istioctl analyze
```

Once you have determined there are no issues with the configuration, get the external url for the application.

```
kubectl get svc istio-ingressgateway -n istio-system -0
```

Your external url will be http://EXTERNAL_IP:80/productpage.  EKS will return a dns name, not an IP.

```
http://********-**************-*********.us-blah-2.elb.amazonaws.com:80/productpage
```

Confirm you can connect.

## Deploy Datadog agent

The datadog-values.yaml included in this repo enables infra, tracing, process, and log collection from all containers.

Create K8s secret using your [API Key](https://app.datadoghq.com/account/settings#api).

```
kubectl create secret generic datadog-agent --from-literal='api-key=<YOUR_API_KEY>' --namespace="default"
```

Install the Agent using helm.

```
helm install datadog-agent -f datadog-values.yaml datadog/datadog --set targetSystem=linux
```

# Appendix

## Helpful commands

upgrading istio after adding tracing settings or tags, etc

```
istioctl upgrade -f /path/to/your/new/manifest.yaml -y
```

upgrading datadog-agent after modifying datadog-valus.yaml

```
helm upgrade datadog-agent -f datadog-values.yaml datadog/datadog --set targetSystem=linux
```

rollout new deployments after making changes to istio

```
kubectl rollout restart deployment --namespace default
```
