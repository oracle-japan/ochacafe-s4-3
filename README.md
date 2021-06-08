# ochacafe-s4-3
Ochacafe4 #3 CI/CD最新事情

デモ環境の構築方法、資材置き場です。
OKE + OCIR + GitHub Actions + ArgoCD

## TEKTON

### TEKTON Install

Kubernetesクラスタ構築されていることが前提です。

**コピー&ペースト用**
```
kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

**コマンド結果**
```
$ kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
namespace/tekton-pipelines created
podsecuritypolicy.policy/tekton-pipelines created
clusterrole.rbac.authorization.k8s.io/tekton-pipelines-controller-cluster-access created
clusterrole.rbac.authorization.k8s.io/tekton-pipelines-controller-tenant-access created
clusterrole.rbac.authorization.k8s.io/tekton-pipelines-webhook-cluster-access created
role.rbac.authorization.k8s.io/tekton-pipelines-controller created
role.rbac.authorization.k8s.io/tekton-pipelines-webhook created
role.rbac.authorization.k8s.io/tekton-pipelines-leader-election created
serviceaccount/tekton-pipelines-controller created
serviceaccount/tekton-pipelines-webhook created
clusterrolebinding.rbac.authorization.k8s.io/tekton-pipelines-controller-cluster-access created
clusterrolebinding.rbac.authorization.k8s.io/tekton-pipelines-controller-tenant-access created
clusterrolebinding.rbac.authorization.k8s.io/tekton-pipelines-webhook-cluster-access created
rolebinding.rbac.authorization.k8s.io/tekton-pipelines-controller created
rolebinding.rbac.authorization.k8s.io/tekton-pipelines-webhook created
rolebinding.rbac.authorization.k8s.io/tekton-pipelines-controller-leaderelection created
rolebinding.rbac.authorization.k8s.io/tekton-pipelines-webhook-leaderelection created
customresourcedefinition.apiextensions.k8s.io/clustertasks.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/conditions.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/pipelines.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/pipelineruns.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/pipelineresources.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/runs.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/tasks.tekton.dev created
customresourcedefinition.apiextensions.k8s.io/taskruns.tekton.dev created
secret/webhook-certs created
validatingwebhookconfiguration.admissionregistration.k8s.io/validation.webhook.pipeline.tekton.dev created
mutatingwebhookconfiguration.admissionregistration.k8s.io/webhook.pipeline.tekton.dev created
validatingwebhookconfiguration.admissionregistration.k8s.io/config.webhook.pipeline.tekton.dev created
clusterrole.rbac.authorization.k8s.io/tekton-aggregate-edit created
clusterrole.rbac.authorization.k8s.io/tekton-aggregate-view created
configmap/config-artifact-bucket created
configmap/config-artifact-pvc created
configmap/config-defaults created
configmap/feature-flags created
configmap/config-leader-election created
configmap/config-logging created
configmap/config-observability created
configmap/config-registry-cert created
deployment.apps/tekton-pipelines-controller created
service/tekton-pipelines-controller created
horizontalpodautoscaler.autoscaling/tekton-pipelines-webhook created
deployment.apps/tekton-pipelines-webhook created
service/tekton-pipelines-webhook created
```

以下2つのPodがRunningとなれば完了です。

**コピー&ペースト用**
```
kubectl get pods --namespace tekton-pipelines
```

**コマンド結果**
```
$ kubectl get pods --namespace tekton-pipelines
NAME                                           READY   STATUS    RESTARTS   AGE
tekton-pipelines-controller-5cf4d4867f-cw4kt   1/1     Running   0          51s
tekton-pipelines-webhook-57bfb4b4d7-bj85t      1/1     Running   0          49s
```

### TEKTON Dashboard Install

**コピー&ペースト用**
```
kubectl apply --filename https://storage.googleapis.com/tekton-releases/dashboard/latest/tekton-dashboard-release.yaml
```

**コマンド結果**
```
$ kubectl apply --filename https://storage.googleapis.com/tekton-releases/dashboard/latest/tekton-dashboard-release.yaml
customresourcedefinition.apiextensions.k8s.io/extensions.dashboard.tekton.dev created
serviceaccount/tekton-dashboard created
clusterrole.rbac.authorization.k8s.io/tekton-dashboard-backend created
clusterrole.rbac.authorization.k8s.io/tekton-dashboard-dashboard created
clusterrole.rbac.authorization.k8s.io/tekton-dashboard-extensions created
clusterrole.rbac.authorization.k8s.io/tekton-dashboard-pipelines created
clusterrole.rbac.authorization.k8s.io/tekton-dashboard-tenant created
clusterrole.rbac.authorization.k8s.io/tekton-dashboard-triggers created
clusterrolebinding.rbac.authorization.k8s.io/tekton-dashboard-backend created
service/tekton-dashboard created
deployment.apps/tekton-dashboard created
rolebinding.rbac.authorization.k8s.io/tekton-dashboard-pipelines created
rolebinding.rbac.authorization.k8s.io/tekton-dashboard-dashboard created
rolebinding.rbac.authorization.k8s.io/tekton-dashboard-triggers created
clusterrolebinding.rbac.authorization.k8s.io/tekton-dashboard-tenant created
clusterrolebinding.rbac.authorization.k8s.io/tekton-dashboard-extensions created
```

`Service`のタイプを`LoadBalancer`に変更して、外部からアクセスできるようにします。

**コピー&ペースト用**
```
kubectl patch service tekton-dashboard -n tekton-pipelines -p '{"spec": {"type": "LoadBalancer"}}'
```

```
$ kubectl patch service tekton-dashboard -n tekton-pipelines -p '{"spec": {"type": "LoadBalancer"}}'
service/tekton-dashboard patched
```

`EXTERNAL-IP`を確認します。ここでは、コマンド結果から「140.83.50.157:9097」にブラウザでアクセスします。

**コピー&ペースト用**
```
kubectl get svc -n tekton-pipelines
```

**コマンド結果**
```
$ kubectl get svc -n tekton-pipelines
NAME                          TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                              AGE
tekton-dashboard              LoadBalancer   10.96.211.23    140.83.50.157   9097:30259/TCP                       109s
tekton-pipelines-controller   ClusterIP      10.96.111.154   <none>          9090/TCP,8080/TCP                    3m46s
tekton-pipelines-webhook      ClusterIP      10.96.124.31    <none>          9090/TCP,8008/TCP,443/TCP,8080/TCP   3m45s
```

### TEKTON CLI Install

**コピー&ペースト用**
```
curl -LO https://github.com/tektoncd/cli/releases/download/v0.18.0/tkn_0.18.0_Linux_x86_64.tar.gz
```

OCI Cloud Shellの場合は、以下の手順を実施します。それ以外は、任意の場所に格納して、パスを通してください。

**コピー&ペースト用**
```
cd
```

**コピー&ペースト用**
```
mkdir bin
```

**コピー&ペースト用**
```
tar -xvzf tkn_0.18.0_Linux_x86_64.tar.gz
```

**コマンド結果**
```
$ tar -xvzf tkn_0.18.0_Linux_x86_64.tar.gz
LICENSE
README.md
tkn
```

**コピー&ペースト用**
```
mv tkn bin
```

パスを通します。`$home`はご自身のホームディレクトリになります。

**コピー&ペースト用**
```
export PATH="$PATH:/home/$home/bin"
```

不要ファイルを削除します。

**コピー&ペースト用**
```
rm -rf LICENSE README.md tkn_0.18.0_Linux_x86_64.tar.gz
```

tknコマンドを実行してバージョンを確認します。

**コピー&ペースト用**
```
tkn version
```

**コマンド結果**
```
$ tkn version
Client version: 0.18.0
Pipeline version: v0.24.1
Dashboard version: v0.17.0
```

### TEKTONパイプライン構築

本リポジトリにある`tekton`ディレクトリ内にある01～07のマニフェストファイルを`kubectl apply`コマンドを実行します。

**コピー&ペースト用**
```
cd tekton
```

`annotations`のULR、`username`、`password`は、ご自身のコンテナレジストリの情報を入力してください。OCIRの場合は、各リージョンは[こちら](https://docs.oracle.com/ja-jp/iaas/Content/Registry/Concepts/registryprerequisites.htm)で確認してください。


**コピー&ペースト用**
```
cat 01-secret-serviceaccount.yaml
```

**コマンド結果**
```
$ cat 01-secret-serviceaccount.yaml
apiVersion: v1
kind: Secret
metadata:
  name: basic-user-pass
  annotations:
    tekton.dev/docker-0: https://xxxxxxxxxxxxxx
type: kubernetes.io/basic-auth
stringData:
  username: xxxxxxxxxx
  password: xxxxxxxxxx
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: build-bot
secrets:
  - name: basic-user-pass
```

**コピー&ペースト用**
```
kubectl apply -f 01-secret-serviceaccount.yaml
```

**コマンド結果**
```**コマンド結果**
$ kubectl apply -f 01-secret-serviceaccount.yaml
secret/basic-user-pass created
serviceaccount/build-bot created
```

`value`には、`git clone`するご自身のgitリポジトリを指定します。

**コピー&ペースト用**
```
cat 02-gitPipelineResource.yaml
```

**コマンド結果**
```**コマンド結果**
$ cat 02-gitPipelineResource.yaml
apiVersion: tekton.dev/v1alpha1
kind: PipelineResource
metadata:
  name: hello-world-go-git
spec:
  type: git
  params:
    - name: revision
      value: main
    - name: url
      value: xxxxxxxxxx
```

**コピー&ペースト用**
```
kubectl apply -f 02-gitPipelineResource.yaml
```

**コマンド結果**
```
$ kubectl apply -f 02-gitPipelineResource.yaml
pipelineresource.tekton.dev/hello-world-go-git created
```

`value`には、ご自身のイメージリポジトリのパスを指定します。

**コピー&ペースト用**
```
cat 03-imagePipelineResource.yaml
```

**コマンド結果**
```
$ cat 03-imagePipelineResource.yaml
apiVersion: tekton.dev/v1alpha1
kind: PipelineResource
metadata:
  name: hello-world-go-image
spec:
  type: image
  params:
    - name: url
      value: xxxxxxxxx/hello-world-go-app
```

**コピー&ペースト用**
```
kubectl apply -f 03-imagePipelineResource.yaml
```

**コマンド結果**
```
$ kubectl apply -f 03-imagePipelineResource.yaml
pipelineresource.tekton.dev/hello-world-go-image created
```

**コピー&ペースト用**
```
kubectl apply -f 04-task.yaml
```

**コマンド結果**
```
$ kubectl apply -f 04-task.yaml
task.tekton.dev/build-and-push created
```

**コピー&ペースト用**
```
kubectl apply -f 05-taskrun.yaml
```

**コマンド結果**
```
$ kubectl apply -f 05-taskrun.yaml
taskrun.tekton.dev/build-push-task-run created
```

**コピー&ペースト用**
```
kubectl apply -f 06-pipeline.yaml
```

**コマンド結果**
```
$ kubectl apply -f 06-pipeline.yaml
pipeline.tekton.dev/build-push-pipeline created
```

**コピー&ペースト用**
```
kubectl apply -f 07-pipelinerun.yaml
```

**コマンド結果**
```
$ kubectl apply -f 07-pipelinerun.yaml
pipelinerun.tekton.dev/build-push-pr created
```

TEKTON Dashboardで状況を確認できます。以下コマンドの実行でも確認できます。

**コピー&ペースト用**
```
tkn pipelinerun logs build-push-pr
```

**コマンド結果**
```
$ tkn pipelinerun logs build-push-pr
[build-and-push : git-source-docker-source-6cb98] {"level":"info","ts":1622703291.5306501,"caller":"git/git.go:169","msg":"Successfully cloned https://xxxxxxxxxx/xxxxxxxxx/xxxxxxxxxx.git @ 600bbea2560177605044546bcddcc3709bf0bb27 (grafted, HEAD, origin/main) in path /workspace/docker-source"}
[build-and-push : git-source-docker-source-6cb98] {"level":"info","ts":1622703291.544796,"caller":"git/git.go:207","msg":"Successfully initialized and updated submodules in path /workspace/docker-source"}

[build-and-push : build-and-push] E0603 06:54:51.936857      13 aws_credentials.go:77] while getting AWS credentials NoCredentialProviders: no valid providers in chain. Deprecated.
[build-and-push : build-and-push]       For verbose messaging see aws.Config.CredentialsChainVerboseErrors
[build-and-push : build-and-push] INFO[0000] Resolved base name golang:1.16 to builder    
[build-and-push : build-and-push] INFO[0000] Retrieving image manifest golang:1.16        
[build-and-push : build-and-push] INFO[0000] Retrieving image golang:1.16 from registry index.docker.io 
[build-and-push : build-and-push] INFO[0002] Retrieving image manifest gcr.io/distroless/base 
[build-and-push : build-and-push] INFO[0002] Retrieving image gcr.io/distroless/base from registry gcr.io 
[build-and-push : build-and-push] INFO[0003] Built cross stage deps: map[0:[/gitops-go-app]] 
[build-and-push : build-and-push] INFO[0003] Retrieving image manifest golang:1.16        
[build-and-push : build-and-push] INFO[0003] Returning cached image manifest              
[build-and-push : build-and-push] INFO[0003] Executing 0 build triggers                   
[build-and-push : build-and-push] INFO[0003] Unpacking rootfs as cmd COPY ./main.go ./ requires it. 
[build-and-push : build-and-push] INFO[0020] COPY ./main.go ./                            
[build-and-push : build-and-push] INFO[0020] Taking snapshot of files...                  
[build-and-push : build-and-push] INFO[0020] RUN go build -o /gitops-go-app ./main.go     
[build-and-push : build-and-push] INFO[0020] Taking snapshot of full filesystem...        
[build-and-push : build-and-push] INFO[0038] cmd: /bin/sh                                 
[build-and-push : build-and-push] INFO[0038] args: [-c go build -o /gitops-go-app ./main.go] 
[build-and-push : build-and-push] INFO[0038] Running: [/bin/sh -c go build -o /gitops-go-app ./main.go] 
[build-and-push : build-and-push] INFO[0038] Taking snapshot of full filesystem...        
[build-and-push : build-and-push] INFO[0039] Saving file gitops-go-app for later use      
[build-and-push : build-and-push] INFO[0039] Deleting filesystem...                       
[build-and-push : build-and-push] INFO[0039] Retrieving image manifest gcr.io/distroless/base 
[build-and-push : build-and-push] INFO[0039] Returning cached image manifest              
[build-and-push : build-and-push] INFO[0039] Executing 0 build triggers                   
[build-and-push : build-and-push] INFO[0039] Unpacking rootfs as cmd COPY --from=builder /gitops-go-app /. requires it. 
[build-and-push : build-and-push] INFO[0041] EXPOSE 8080                                  
[build-and-push : build-and-push] INFO[0041] cmd: EXPOSE                                  
[build-and-push : build-and-push] INFO[0041] Adding exposed port: 8080/tcp                
[build-and-push : build-and-push] INFO[0041] COPY --from=builder /gitops-go-app /.        
[build-and-push : build-and-push] INFO[0041] Taking snapshot of files...                  
[build-and-push : build-and-push] INFO[0041] ENTRYPOINT ["./gitops-go-app"]               
[build-and-push : build-and-push] INFO[0041] Pushing image to xxxxxxxxx/xxxxxxxxx/xxxxxxxxx/hello-world-go-app 
[build-and-push : build-and-push] INFO[0042] Pushed image to 1 destinations               

[build-and-push : image-digest-exporter-7r87w] {"severity":"INFO","timestamp":"2021-06-03T06:55:34.718982043Z","caller":"logging/config.go:116","message":"Successfully created the logger."}
[build-and-push : image-digest-exporter-7r87w] {"severity":"INFO","timestamp":"2021-06-03T06:55:34.719032609Z","caller":"logging/config.go:117","message":"Logging level set to: info"}
[build-and-push : image-digest-exporter-7r87w] {"severity":"INFO","timestamp":"2021-06-03T06:55:34.719137347Z","caller":"imagedigestexporter/main.go:59","message":"No index.json found for: builtImage","commit":"7ca5d61"}
```

## GitHub Actions

### code & config リポジトリの作成

ご自身のGitHubアカウントを利用して、codeとconfigのリポジトリを作成します。
codeには本リポジトリの`code-repo`、configには`config-repo`内にある資材を格納します。

### Personal Access Tokensの作成

1.ヘッダーメニューにあるアカウントアイコンのプルダウンメニューを開いて「Settings」を選択します。
2.左メニュー「Developer settings」-「Personal access tokens」を選択します。
3.「Generate new token」ボタンをクリックして、「note」に任意名、「repo,workflow,admin:org, delete_repo 」にチェックを入れて、「Generate token」ボタンをクリックします。
4.トークンが生成されるので、エディタなどにコピー&ペーストして保存します。

### Secretの作成

codeリポジトリで利用するGitHub ActionsのYAMLファイル内にあるSecretの設定を行います。

1.GitHub上部メニュー「Settings」を選択して、左メニュー「Secrets」を選択します。
2.右上の「New repository secret」をクリックして、「Name」に「DOCKER_USERNAME」と入力してご自身のコンテナレジストリのアカウント名を設定します。
3.右上の「New repository secret」をクリックして、「Name」に「DOCKER_PASSWORD」と入力してご自身のコンテナレジストリのパスワードを設定します。
4.右上の「New repository secret」をクリックして、「Name」に「USERNAME」と入力してご自身のGitHubアカウント名を設定します。
5.右上の「New repository secret」をクリックして、「Name」に「GH_PASSWORD」と入力してご自身のGitHubパスワードを設定します。
6.右上の「New repository secret」をクリックして、「Name」に「EMAIL」と入力してご自身のGitHubに登録しているメールアドレスを設定します。
7.右上の「New repository secret」をクリックして、「Name」に「PERSONAL_ACCESS_TOKEN」と入力して事前に作成したトークンを設定します。

codeリポジトリのmain.yml内の「xxxxxxxxxx」となっている箇所には、ご自身のイメージレジストリを指定するのを忘れないでください。

## ArgoCD

### ArgoCD Install

**コピー&ペースト用**
```
helm repo add argo https://argoproj.github.io/argo-helm
```

**コマンド結果**
```
$ helm repo add argo https://argoproj.github.io/argo-helm
"argo" has been added to your repositories
```

**コピー&ペースト用**
```
helm repo update
```

**コマンド結果**
```
$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "argo" chart repository
Update Complete. ⎈Happy Helming!⎈
```

**コピー&ペースト用**
```
helm search repo argocd
```

**コマンド結果**
```
$ helm search repo argocd
NAME                            CHART VERSION   APP VERSION     DESCRIPTION                                       
argo/argocd-applicationset      0.1.6           v0.1.0          A Helm chart for installing ArgoCD ApplicationSet 
argo/argocd-notifications       1.4.0           1.1.1           A Helm chart for ArgoCD notifications, an add-o...
argo/argo-cd                    3.6.6           2.0.3           A Helm chart for ArgoCD, a declarative, GitOps ...
```

**コピー&ペースト用**
```
kubectl create namespace argocd
```

**コマンド結果**
```
$ kubectl create namespace argocd
namespace/argocd created
```

**コピー&ペースト用**
```
kubectl get ns argocd
```

**コマンド結果**
```
$ kubectl get ns argocd
NAME     STATUS   AGE
argocd   Active   63s
```

**コピー&ペースト用**
```
helm install argo-cd -n argocd argo/argo-cd --version 3.6.6
```

**コマンド結果**
```
$ helm install argo-cd -n argocd argo/argo-cd --version 3.6.6
manifest_sorter.go:192: info: skipping unknown hook: "crd-install"
manifest_sorter.go:192: info: skipping unknown hook: "crd-install"
NAME: argo-cd
LAST DEPLOYED: Fri Jun  4 07:34:25 2021
NAMESPACE: argocd
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
In order to access the server UI you have the following options:

1. kubectl port-forward service/argo-cd-argocd-server -n argocd 8080:443

    and then open the browser on http://localhost:8080 and accept the certificate

2. enable ingress in the values file `server.ingress.enabled` and either
      - Add the annotation for ssl passthrough: https://github.com/argoproj/argo-cd/blob/master/docs/operator-manual/ingress.md#option-1-ssl-passthrough
      - Add the `--insecure` flag to `server.extraArgs` in the values file and terminate SSL at your ingress: https://github.com/argoproj/argo-cd/blob/master/docs/operator-manual/ingress.md#option-2-multiple-ingress-objects-and-hosts


After reaching the UI the first time you can login with username: admin and the random password generated during the installation. You can find the password by running:

kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

(You should delete the initial secret afterwards as suggested by the Getting Started Guide: https://github.com/argoproj/argo-cd/blob/master/docs/getting_started.md#4-login-using-the-cli)
```

**コピー&ペースト用**
```
kubectl get pods,services -n argocd
```

**コマンド結果**
```
$ kubectl get pods,services -n argocd
NAME                                                         READY   STATUS    RESTARTS   AGE
pod/argo-cd-argocd-application-controller-5d559b8cfc-szf5z   1/1     Running   0          64s
pod/argo-cd-argocd-dex-server-74b56bbbc4-2dpfd               1/1     Running   0          64s
pod/argo-cd-argocd-redis-567c5bb96-ljvmw                     1/1     Running   0          64s
pod/argo-cd-argocd-repo-server-76449ddf69-f5tdh              1/1     Running   0          64s
pod/argo-cd-argocd-server-66f87579f-52tcr                    1/1     Running   0          64s

NAME                                            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
service/argo-cd-argocd-application-controller   ClusterIP   10.96.126.122   <none>        8082/TCP            64s
service/argo-cd-argocd-dex-server               ClusterIP   10.96.4.52      <none>        5556/TCP,5557/TCP   64s
service/argo-cd-argocd-redis                    ClusterIP   10.96.84.111    <none>        6379/TCP            64s
service/argo-cd-argocd-repo-server              ClusterIP   10.96.174.128   <none>        8081/TCP            64s
service/argo-cd-argocd-server                   ClusterIP   10.96.101.57    <none>        80/TCP,443/TCP      64s
```

**コピー&ペースト用**
```
kubectl patch service argo-cd-argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

**コマンド結果**
```
$ kubectl patch service argo-cd-argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
service/argo-cd-argocd-server patched
```

**コピー&ペースト用**
```
kubectl get service -n argocd
```

**コマンド結果**
```
$ kubectl get service -n argocd
NAME                                    TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                      AGE
argo-cd-argocd-application-controller   ClusterIP      10.96.126.122   <none>          8082/TCP                     5m52s
argo-cd-argocd-dex-server               ClusterIP      10.96.4.52      <none>          5556/TCP,5557/TCP            5m52s
argo-cd-argocd-redis                    ClusterIP      10.96.84.111    <none>          6379/TCP                     5m52s
argo-cd-argocd-repo-server              ClusterIP      10.96.174.128   <none>          8081/TCP                     5m52s
argo-cd-argocd-server                   LoadBalancer   10.96.101.57    168.138.51.60   80:31859/TCP,443:31707/TCP   5m52s
```

ブラウザを起動して、「35.200.124.246」にアクセスします。

### argocd CLI Install

**コピー&ペースト用**
```
wget https://github.com/argoproj/argo-cd/releases/download/v2.0.3/argocd-linux-amd64
```

**コマンド結果**
```
$ wget https://github.com/argoproj/argo-cd/releases/download/v2.0.3/argocd-linux-amd64
--2021-06-04 07:42:34--  https://github.com/argoproj/argo-cd/releases/download/v2.0.3/argocd-linux-amd64
Resolving github.com (github.com)... 140.82.112.4
Connecting to github.com (github.com)|140.82.112.4|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://github-releases.githubusercontent.com/120896210/9d57e880-bf13-11eb-9dcc-1a6e5fd64fe3?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20210604%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20210604T074234Z&X-Amz-Expires=300&X-Amz-Signature=47360d3f3a8f11451a4d97d7165283ab4942559530df1d896c4e64b3e019058c&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=120896210&response-content-disposition=attachment%3B%20filename%3Dargocd-linux-amd64&response-content-type=application%2Foctet-stream [following]
--2021-06-04 07:42:34--  https://github-releases.githubusercontent.com/120896210/9d57e880-bf13-11eb-9dcc-1a6e5fd64fe3?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAIWNJYAX4CSVEH53A%2F20210604%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20210604T074234Z&X-Amz-Expires=300&X-Amz-Signature=47360d3f3a8f11451a4d97d7165283ab4942559530df1d896c4e64b3e019058c&X-Amz-SignedHeaders=host&actor_id=0&key_id=0&repo_id=120896210&response-content-disposition=attachment%3B%20filename%3Dargocd-linux-amd64&response-content-type=application%2Foctet-stream
Resolving github-releases.githubusercontent.com (github-releases.githubusercontent.com)... 185.199.108.154, 185.199.109.154, 185.199.110.154, ...
Connecting to github-releases.githubusercontent.com (github-releases.githubusercontent.com)|185.199.108.154|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 89451754 (85M) [application/octet-stream]
Saving to: ‘argocd-linux-amd64’

100%[=======================================================================================================>] 89,451,754  98.9MB/s   in 0.9s   

2021-06-04 07:42:35 (98.9 MB/s) - ‘argocd-linux-amd64’ saved [89451754/89451754]
```

**コピー&ペースト用**
```
$ mv argocd-linux-amd64 argocd
```

**コピー&ペースト用**
```
$ chmod +x argocd
```

tknコマンド同じように`bin`ディレクトリに格納します。

**コピー&ペースト用**
```
$ mv argocd bin
```

**コピー&ペースト用**
```
argocd version
```

**コマンド結果**
```
$ argocd version
argocd: v2.0.3+8d2b13d
  BuildDate: 2021-05-27T17:38:37Z
  GitCommit: 8d2b13d733e1dff7d1ad2c110ed31be4804406e2
  GitTreeState: clean
  GoVersion: go1.16
  Compiler: gc
  Platform: linux/amd64
FATA[0000] Failed to establish connection to 140.83.57.157:443: EOF
```

ArgoCD WebUIのURLを取得します。

**コピー&ペースト用**
```
export ARGOCD_EXTERNAL_IP=$(kubectl get svc argo-cd-argocd-server -n argocd -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
```

WebUIの初期パスワードを生成します。

**コピー&ペースト用**
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

**コマンド結果**
```
$ kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
SGFxgfUJdKaTiqXA
```

argocdコマンドでログインして、初期パスワードを`argocd`に変更します。

**コピー&ペースト用**
```
argocd --insecure login ${ARGOCD_EXTERNAL_IP} --username admin
```

**コマンド結果**
```
$ argocd --insecure login ${ARGOCD_EXTERNAL_IP} --username admin
Password: 
'admin:login' logged in successfully
Context '168.138.51.60' updated
```

初期パスワードを入力して、新パスワードとして`argocd`を二回入力します。

**コピー&ペースト用**
```
argocd  account update-password --account admin
```

**コマンド結果**
```
$ argocd  account update-password --account admin
*** Enter current password: 
*** Enter new password: 
*** Confirm new password: 
Password updated
Context '168.138.51.60' updated
```

ブラウザで「168.138.51.60」にアクセスして、以下の情報を入力してログインします。

Username:admin
Password:argocd

![ArgoCD WebUI](image/ochacafe-s4-3-01.png "ArgoCD WebUI")

`NEW APP`ボタンをクリックします。

![NEW APP](image/ochacafe-s4-3-02.png "NEW APP")


ご自身のConfgリポジトリ（git）に本リポジトリにある`config-repo`にある`gitops-helm`と`.github`を格納した状態で、
以下を設定します。

- GENERAL
    - Application Name:gitops-go-app
    - Project:default
    - SYNC POLICY:Automatic
- SOURCE
    - Repository URL:ご自身のConfigリポジトリ
    - Path:gitops-helm
- DESTINATION
    - Cluster URL:https://kubernetes.default.svc
    - Namespace:default

HELM
VALUES FILES:values.yaml

![Config](image/ochacafe-s4-3-03.png "Config")

Syncしていることを確認します。この表示自体をクリックすると全体画面に遷移します。

![Sync](image/ochacafe-s4-3-04.png "Sync")

![Cluster Status](image/ochacafe-s4-3-05.png "Cluster Status")

## Gatekeeper

### Gatekeeper Install

**コピー&ペースト用**
```
kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/v3.1.3/deploy/gatekeeper.yaml
```

**コマンド結果**
```
$ kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/v3.1.3/deploy/gatekeeper.yaml
namespace/gatekeeper-system created
customresourcedefinition.apiextensions.k8s.io/configs.config.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/constraintpodstatuses.status.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/constrainttemplatepodstatuses.status.gatekeeper.sh created
customresourcedefinition.apiextensions.k8s.io/constrainttemplates.templates.gatekeeper.sh created
serviceaccount/gatekeeper-admin created
role.rbac.authorization.k8s.io/gatekeeper-manager-role created
clusterrole.rbac.authorization.k8s.io/gatekeeper-manager-role created
rolebinding.rbac.authorization.k8s.io/gatekeeper-manager-rolebinding created
clusterrolebinding.rbac.authorization.k8s.io/gatekeeper-manager-rolebinding created
secret/gatekeeper-webhook-server-cert created
service/gatekeeper-webhook-service created
deployment.apps/gatekeeper-audit created
deployment.apps/gatekeeper-controller-manager created
validatingwebhookconfiguration.admissionregistration.k8s.io/gatekeeper-validating-webhook-configuration created
```

本リポジトリにある`gatekeeper`ディレクトリ内にある各マニフェストファイルを`kubectl apply`コマンドで適用します。

**コピー&ペースト用**
```
cd
```

**コピー&ペースト用**
```
cd gatekeeper
```

**コピー&ペースト用**
```
cat constrainttemplate.yaml
```

**コマンド結果**
```
$ cat constrainttemplate.yaml
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: notlatestimage
spec:
  crd:
    spec:
      names:
        kind: NotLatestImage
        listKind: NotLatestImageList
        plural: notlatestimages
        singular: notlatestimage
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package notlatestimage

        violation[{"msg": msg}]{
          input.review.object.kind == "Pod"
          imagetag := input.review.object.spec.containers[_].image
          endswith(imagetag,"latest")
          msg := "Can't use image of latest tag !!"
        }
```

**コピー&ペースト用**
```
kubectl apply -f constrainttemplate.yaml
```

**コマンド結果**
```
$ kubectl apply -f constrainttemplate.yaml
constrainttemplate.templates.gatekeeper.sh/notlatestimage created
```

**コピー&ペースト用**
```
cat constraints.yaml
```

**コマンド結果**
```
$ cat constraints.yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: NotLatestImage
metadata:
  name: notlatestimage
spec:
  match:
    kinds:
    - apiGroups: [""]
      kinds: ["Pod"]
```

**コピー&ペースト用**
```
kubectl apply -f constraints.yaml
```

**コマンド結果**
```
$ kubectl apply -f constraints.yaml
notlatestimage.constraints.gatekeeper.sh/notlatestimage created
```

**コピー&ペースト用**
```
cat banlataest.yaml
```

**コマンド結果**
```
$ cat banlataest.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-nginx-latest
spec:
  containers:
  - name: nginx-latesttag
    image: nginx:latest
```

**コピー&ペースト用**
```
kubectl apply -f banlataest.yaml
```

**コマンド結果**
```
$ kubectl apply -f banlataest.yaml
Error from server ([denied by notlatestimage] Can't use image of latest tag !!): error when creating "banlataest.yaml": admission webhook "validation.gatekeeper.sh" denied the request: [denied by notlatestimage] Can't use image of latest tag !!
```

