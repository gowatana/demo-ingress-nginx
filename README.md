# ingress demo


Download kubectl

```
$ SCP=192.168.21.128
$ curl -k -O https://$SCP/wcp/plugin/linux-amd64/vsphere-plugin.zip
$ unzip ./vsphere-plugin.zip
```

set env

```
$ . env_kubectl
$ kubectl version --short --client
```

create tkc

```
$ kubectl vsphere login --server=192.168.21.128 --insecure-skip-tls-verify
$ kubectl config use-context lab-ns-41
$ kubectl apply -f 01_tkc/tanzu-cluster-41.yml
```

connect to tkc

```
$ sh ./01_tkc/login_tanzu-cluster-41.sh
$ kubectl config use-context tanzu-cluster-41
$ kubectl get nodes
```

helm install

```
$ curl -OL https://get.helm.sh/helm-v3.5.2-linux-amd64.tar.gz
$ tar zxvf ./helm-v3.5.2-linux-amd64.tar.gz
$ cp ./linux-amd64/helm ./bin/
$ helm version --short
```

ingress-nginx install

```
$ kubectl create ns ingress-nginx
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
$ helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx
$ kubectl -n ingress-nginx get all
$ kubectl -n ingress-nginx get services
$ kubectl -n ingress-nginx get ingress
```

example:

```
Tanzu$ helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx
NAME: ingress-nginx
LAST DEPLOYED: Wed Feb 24 23:42:16 2021
NAMESPACE: ingress-nginx
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
It may take a few minutes for the LoadBalancer IP to be available.
You can watch the status by running 'kubectl --namespace ingress-nginx get services -o wide -w ingress-nginx-controller'

An example Ingress that makes use of the controller:

  apiVersion: networking.k8s.io/v1beta1
  kind: Ingress
  metadata:
    annotations:
      kubernetes.io/ingress.class: nginx
    name: example
    namespace: foo
  spec:
    rules:
      - host: www.example.com
        http:
          paths:
            - backend:
                serviceName: exampleService
                servicePort: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
        - hosts:
            - www.example.com
          secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls
Tanzu$
```

create deployment / service / ingress

```
$ kubectl create ns ns-web-01
$ kubectl -n ns-web-01 apply -f 03_web/svc-web-a.yml
$ kubectl -n ns-web-01 apply -f 03_web/svc-web-b.yml
$ kubectl -n ns-web-01 get all
$ kubectl -n ns-web-01 apply -f 03_web/ingress_web-example.yml
$ kubectl -n ns-web-01 get ingress
$ kubectl -n ns-web-01 get all
```


EOF
