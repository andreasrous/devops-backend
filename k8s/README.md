# Container Registry (github packages)

## Github Packages

- Create personal access token (settings --> Developer settings -- > Personal Access Tokens), select classic
- Select read:packages
- Save the token to a file
- To see packages, go to your github profile and select tab Packages
- Tag an image

```bash
docker build -t ghcr.io/<GITHUB-USERNAME>/ds-spring:latest -f nonroot.Dockerfile .
```

- Login to docker registry

```bash
cat ~/github-image-repo.txt | docker login ghcr.io -u <GITHUB-USERNAME> --password-stdin
```

- Push image

```bash
docker push ghcr.io/<GITHUB-USERNAME>/ds-spring:latest
```

## Create docker login secret

- Create a .dockerconfig.json file, like this

```json
{
  "auths": {
    "https://ghcr.io": {
      "username": "REGISTRY_USERNAME",
      "password": "REGISTRY_TOKEN",
      "email": "REGISTRY_EMAIL",
      "auth": "BASE_64_BASIC_AUTH_CREDENTIALS"
    }
  }
}
```

- Create <BASE_64_BASIC_AUTH_CREDENTIALS> from the command

```bash
echo -n <USER>:<TOKEN> | base64
```

## Create dockerconfig secret

```bash
kubectl create secret docker-registry registry-credentials --from-file=.dockerconfigjson=k8s/.dockerconfig.json
```

## Install helm

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3

chmod 700 get_helm.sh

./get_helm.sh
```

## Install cert-manager

```bash
helm repo add jetstack https://charts.jetstack.io --force-update

helm install \
cert-manager jetstack/cert-manager \
--namespace cert-manager \
--create-namespace \
--version v1.14.5 \
--set installCRDs=true
```

## Create cert-issuer

Replace email in k8s/cert-issuer/cert-issuer.yaml file and apply

```bash
kubectl apply -f k8s/cert/cert-issuer.yaml
```

# Links

- [cert-manager](https://cert-manager.io/docs/installation/helm/)
- [Troubleshooting cert-manager](https://cert-manager.io/docs/troubleshooting/)
- [Spring boot health probes](https://www.baeldung.com/spring-liveness-readiness-probes)
