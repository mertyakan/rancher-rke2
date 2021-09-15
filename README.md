# rancher-rke2 install single node cewntos7

---
# 1. Install Kubernetes and Set up the RKE2 Server

The **ntp** (Network Time Protocol) package should be installed. This prevents errors with certificate validation that can occur when the time is not synchronized between the client and server.

RKE2 server runs with embedded etcd so you will not need to set up an external datastore to run in HA mode.

> Here is an example of what the RKE2 config file (at /etc/rancher/rke2/config.yaml) would look like if you are following this guide:

```
token: my-shared-secret
tls-san:
  - my-kubernetes-domain.com or ip 

```
> After that you need to run the install command and enable and start rke2:

```
curl -sfL https://get.rke2.io | INSTALL_RKE2_CHANNEL=v1.20 sh -
systemctl enable rke2-server.service
systemctl start rke2-server.service

```
> Follow the logs, if you like

```
journalctl -u rke2-server -f
```

> Wait a bit
```
export KUBECONFIG=/etc/rancher/rke2/rke2.yaml PATH=$PATH:/var/lib/rancher/rke2/bin
kubectl get nodes
```
https://github.com/rancher/rke2

> get helm
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
helm repo add rancher-latest https://releases.rancher.com/server-charts/latest
kubectl create namespace cattle-system

> get cert-manager
The Rancher management server is designed to be secure by default and requires SSL/TLS configuration.

    Note:
    If you want terminate SSL/TLS externally, see TLS termination on an External Load Balancer.

There are three recommended options for the source of the certificate used for TLS termination at the Rancher server:

    Rancher-generated TLS certificate: In this case, you will need to install cert-manager into the cluster. Rancher utilizes cert-manager to issue and maintain its certificates. Rancher will generate a CA certificate of its own, and sign a cert using that CA. cert-manager is then responsible for managing that certificate.
    Let’s Encrypt: The Let’s Encrypt option also uses cert-manager. However, in this case, cert-manager is combined with a special Issuer for Let’s Encrypt that performs all actions (including request and validation) necessary for getting a Let’s Encrypt issued cert. This configuration uses HTTP validation (HTTP-01), so the load balancer must have a public DNS record and be accessible from the internet.
    Bring your own certificate: This option allows you to bring your own public- or private-CA signed certificate. Rancher will use that certificate to secure websocket and HTTPS traffic. In this case, you must upload this certificate (and associated key) as PEM-encoded files with the name tls.crt and tls.key. If you are using a private CA, you must also upload that certificate. This is due to the fact that this private CA may not be trusted by your nodes. Rancher will take that CA certificate, and generate a checksum from it, which the various Rancher components will use to validate their connection to Rancher.

> Install cert-manager
```
# If you have installed the CRDs manually instead of with the `--set installCRDs=true` option added to your Helm install command, you should upgrade your CRD resources before upgrading the Helm chart:
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.5.1/cert-manager.crds.yaml

# Add the Jetstack Helm repository
helm repo add jetstack https://charts.jetstack.io

# Update your local Helm chart repository cache
helm repo update

# Install the cert-manager Helm chart
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.5.1
```
Once you’ve installed cert-manager, you can verify it is deployed correctly by checking the cert-manager namespace for running pods:
```
[root@rancher ~]# kubectl get pods --namespace cert-manager
NAME                                      READY   STATUS    RESTARTS   AGE
cert-manager-56b686b465-6npff             1/1     Running   0          61s
cert-manager-cainjector-75c94654d-8c6b6   1/1     Running   0          61s
cert-manager-webhook-d4fd4f479-pv2d2      1/1     Running   0          61s
```

The default is for Rancher to generate a CA and uses cert-manager to issue the certificate for access to the Rancher server interface.

Because rancher is the default option for ingress.tls.source, we are not specifying ingress.tls.source when running the helm install command.

    Set the hostname to the DNS name you pointed at your load balancer.
    Set the bootstrapPassword to something unique for the admin user.
    If you are installing an alpha version, Helm requires adding the --devel option to the command.
    To install a specific Rancher version, use the --version flag, example: --version 2.3.6
```
helm install rancher rancher-latest/rancher \
  --namespace cattle-system \
  --set hostname=rancher.my.org \
  --set bootstrapPassword=admin
```
Wait for Rancher to be rolled out:
```
kubectl -n cattle-system rollout status deploy/rancher
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out
```




# output

[root@rancher ~]# helm install rancher rancher-latest/rancher \
>   --namespace cattle-system \
>   --set hostname=rke2.rancher.local \
>   --set bootstrapPassword=admin
W0915 20:10:18.101826   26224 warnings.go:70] cert-manager.io/v1beta1 Issuer is deprecated in v1.4+, unavailable in v1.6+; use cert-manager.io/v1 Issuer
NAME: rancher
LAST DEPLOYED: Wed Sep 15 20:10:17 2021
NAMESPACE: cattle-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Rancher Server has been installed.

NOTE: Rancher may take several minutes to fully initialize. Please standby while Certificates are being issued, Containers are started and the Ingress rule comes up.

Check out our docs at https://rancher.com/docs/

If you provided your own bootstrap password during installation, browse to https://rke2.rancher.local to get started.

If this is the first time you installed Rancher, get started by running this command and clicking the URL it generates:

```
echo https://rke2.rancher.local/dashboard/?setup=$(kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}')
```

To get just the bootstrap password on its own, run:

```
kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}{{ "\n" }}'
```

Happy Containering!
[root@rancher ~]# echo https://rke2.rancher.local/dashboard/?setup=$(kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}')
https://rke2.rancher.local/dashboard/?setup=admin
[root@rancher ~]# kubectl get secret --namespace cattle-system bootstrap-secret -o go-template='{{.data.bootstrapPassword|base64decode}}{{ "\n" }}'
admin

[root@rancher ~]# kubectl -n cattle-system rollout status deploy/rancher
deployment "rancher" successfully rolled out
[root@rancher ~]# 

> get https://rke2.rancher.local

![This is an image](http://mertyakan.com/wp-content/uploads/2021/09/Screenshot-from-2021-09-15-20-43-23.png)





























