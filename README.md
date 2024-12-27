# SUSE Observability (StackState) deployment
This repository provide the steps to deploy SUSE Observability (StackState) manually using Helm. This is for testing purpose and uses no high-availability option.

## Step 01 - Creating a Certificate
### Self-Signed Certificate (Opitional)
Create a tls-secret using the base_url for observability

```bash
openssl genrsa -out ca.key 2048
```
```bash
openssl req -x509 \                
  -new -nodes  \
  -days 365 \
  -key ca.key \
  -out ca.crt \
  -subj "/CN=rralab-obs.do.support.rancher.space"
```

```yaml
kubectl create secret tls tls-secret \ 
--key ca.key \
--cert ca.crt
```

### Create a valid certificate trusted by a CA
This is a requirement for some StackState functions.

[Free-SSL-Certificates-with-Certmanager-and-Lets-Encrypt](ttps://github.com/rodolfoalmeid/)

## Step 02

Create an Ingress yaml file.

### Ingress example for self-signed certificate

```yaml
ingress:
  enabled: true
  annotations:
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"
  hosts:
    - host: rralab-obs.do.support.rancher.space
  tls:
    - hosts:
        - rralab-obs.do.support.rancher.space
      secretName: tls-secret
```

### Ingress example for valid ssl certificate
Example to be used with Certmanager and Let's Encrypt

```yaml
ingress:
  enabled: true
  annotations:
    nginx.ingress.kubernetes.io/proxy-body-size: "50m"
    kubernetes.io/ingress.class: "nginx"  
  hosts:
    - host: rralab-obs.do.support.rancher.space
      http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service: 
              name: suse-observability-ui
              port: 
                number: 443
  tls:
    - hosts:
      - rralab-obs.do.support.rancher.space
      secretName: observability-tls-secret
```

## Step 03
Run the following command to create the helm files to install observability

Required information:
- license
- baseUrl
- size

```bash
export VALUES_DIR=.
helm template \
  --set license='KE7UD-Q0VDQ-JM76A' \
  --set baseUrl='rralab-obs.do.support.rancher.space' \
  --set sizing.profile='10-nonha' \
  suse-observability-values \
  suse-observability/suse-observability-values --output-dir $VALUES_DIR
```

## Step 04
Run the helm command to install SUSE Observability.

```bash
helm upgrade --install \
  --namespace "suse-observability" \
  --create-namespace \
  --values "ingress.yaml" \
  --values $VALUES_DIR/suse-observability-values/templates/baseConfig_values.yaml \
  --values $VALUES_DIR/suse-observability-values/templates/sizing_values.yaml \  
suse-observability \
suse-observability/suse-observability
```

In my computer the above command did not work. It was required to run the command in a different format but with the same parameters. 
```bash
helm upgrade --install suse-observability --namespace suse-observability --create-namespace suse-observability/suse-observability --values "ingress.yaml" --values $VALUES_DIR/suse-observability-values/templates/baseConfig_values.yaml --values $VALUES_DIR/suse-observability-values/templates/sizing_values.yaml --set global.skipSslValidation=true
```



***
## Agent Installation

```
helm upgrade --install --namespace suse-observability --create-namespace --set-string 'stackstate.apiKey'='h1tgCiuCRB8CRW8kTvMX1HrPEPD25iH2' --set-string 'stackstate.cluster.name'='rralab-cluster1' --set-string 'stackstate.url'='https://rralab-obs.do.support.rancher.space/receiver/stsAgent' suse-observability-agent suse-observability/suse-observability-agent
```
