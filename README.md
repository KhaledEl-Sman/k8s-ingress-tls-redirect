# Ingress App with TLS Termination and HTTPS Redirect

This guide provides detailed instructions to set up an Ingress resource with TLS termination and automatic HTTP to HTTPS redirection for your Kubernetes cluster. This setup ensures secure communication and a better user experience.

## Prerequisites

- A Kubernetes cluster up and running.
- kubectl configured to connect to your cluster.
- OpenSSL installed for generating TLS certificates.

## Step 1: Setting Up NGINX Ingress Controller

If you are using a local Kubernetes cluster with Kind, create the cluster and expose HTTP and HTTPS ports:

```bash
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF
```

Deploy the NGINX Ingress Controller:

```bash
kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/deploy-ingress-nginx.yaml
```

Wait for the Ingress Controller to become ready:

```bash
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

## Step 2: Deploying Pods and Services

Apply the Pod and Service definitions for your applications:

```bash
kubectl apply -f www.yaml
kubectl apply -f api.yaml
kubectl apply -f admin.yaml
```

## Step 3: Editing /etc/hosts for Local Domain Resolution

Add the following entries to your /etc/hosts file for local testing:

```bash
127.0.0.1 www.example.com
127.0.0.1 api.example.com
```

## Step 4: Generating TLS Certificate

Use the following command to generate a self-signed TLS certificate:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=example.com/0=example.com"
```

## Step 5: Creating a TLS Secret

Create a TLS secret in Kubernetes using the generated certificate and key:

```bash
kubectl create secret tls example-tls-secret --cert=tls.crt --key=tls.key
```

## Step 6: Configuring Ingress with TLS and HTTPS Redirect

Apply the Ingress configuration file with TLS and HTTPS redirection:

```bash
kubectl apply -f ingress.yaml
```

The Ingress is configured to redirect HTTP traffic to HTTPS using the following annotation:

```yaml
annotations:
  nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
```

## Step 7: Verifying the Setup

- Verify Ingress resource creation:

```bash
kubectl get ingress
```

- Describe the Ingress for detailed information:

```bash
kubectl describe ingress ingress
```

- Test HTTPS redirection using curl:

```bash
curl -k https://www.example.com
```

## Troubleshooting

- Ensure the TLS secret name in the Ingress file matches the secret created.
- Check if the NGINX Ingress Controller is correctly deployed.
- Review Ingress logs for any errors using:

```bash
kubectl logs -l app.kubernetes.io/name=ingress-nginx -n ingress-nginx
```
