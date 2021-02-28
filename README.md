# ingress-nginx demo

vSphere with Tanzu の、Tanzu Kubernetes Cluster での Ingress デモ。

## 0. スーパーバイザー名前空間の作成

vSphere Client で、lab-ns-51 という名前で作成ずみ。


## 1. Tanzu Kubernetes Cluster（TKC）作成

スーパーバイザー 制御プレーンから、kubectl をダウンロード。

```
$ SCP=192.168.25.128
$ curl -k -O https://$SCP/wcp/plugin/linux-amd64/vsphere-plugin.zip
$ unzip ./vsphere-plugin.zip
```

環境変数を読み込み。（PS1、PATH、KUBECONFIG など）

```
$ . env_kubectl
$ kubectl version --short --client
```

vSphere with Tanzu のスーパーバイザ制御プレーンに接続。

```
$ kubectl vsphere login --server=$SCP --insecure-skip-tls-verify
```

参考： kubectl vsphere login の例。

```
Tanzu$ kubectl vsphere login --server=$SCP --insecure-skip-tls-verify

Username: administrator@vsphere.local
Password:
Logged in successfully.

You have access to the following contexts:
   192.168.25.128
   lab-ns-51

If the context you wish to use is not in this list, you may need to try
logging in again later, or contact your cluster administrator.

To change context, use `kubectl config use-context <workload name>`
Tanzu$
```

Tanzu Kubernetes Cluster（TKC）の作成。

```
$ kubectl --context lab-ns-51 apply -f 01_tkc/tanzu-cluster-51.yml
```

TKC への接続。

```
$ sh ./01_tkc/login_tanzu-cluster-51.sh
$ kubectl config use-context tanzu-cluster-51
$ kubectl get nodes
```

PodSecurityPolicy を ClusterRoleBindings で割り当て。

```
$ kubectl apply 01_tkc/psp_ClusterRoleBindings.yml
```

## 2. Ingress コントローラ（ingress-nginx）のインストール

helm のインストール。

```
$ curl -OL https://get.helm.sh/helm-v3.5.2-linux-amd64.tar.gz
$ tar zxvf ./helm-v3.5.2-linux-amd64.tar.gz
$ cp ./linux-amd64/helm ./bin/
$ helm version --short
```

helm での、ingress-nginx のインストール。

```
$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

$ kubectl create ns ingress-nginx
$ helm install ingress-nginx ingress-nginx/ingress-nginx -n ingress-nginx
```

確認。

```
$ kubectl -n ingress-nginx get all
$ kubectl -n ingress-nginx get services
```

参考： helm install の例。

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

## 3. Ingress リソース作成のデモ

Kubernetes リソースを作成する。deployment / service / ingress

Namespace 作成（ns-web-01）

```
$ NS=ns-web-01
$ kubectl create ns $NS
```

deployment / service の作成。

```
$ kubectl -n $NS apply -f 03_web/svc-web-a.yml
$ kubectl -n $NS apply -f 03_web/svc-web-b.yml
$ kubectl -n $NS get all
```

ingress リソースの作成。

```
$ kubectl -n $NS apply -f 03_web/ingress_web-example.yml
$ kubectl -n $NS get ingress
$ kubectl -n $NS get all
```

curl / web ブラウザから確認する。パスは /a と /b 。

## 4. TKC の削除。

TKC を削除する。

```
$ kubectl --context lab-ns-51 delete -f 01_tkc/tanzu-cluster-51.yml
```

EOF
