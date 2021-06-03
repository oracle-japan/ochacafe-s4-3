# ochacafe-s4-3
Ochacafe4 #3 CI/CD最新事情

デモ環境の構築方法、資材置き場です。

## TEKTON

### TEKTON Install

Kubernetesクラスタ構築されていることが前提です。

```kubectlコマンド
kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

```コマンド結果
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

```kubectlコマンド
kubectl get pods --namespace tekton-pipelines
```

```コマンド結果
NAME                                           READY   STATUS    RESTARTS   AGE
tekton-pipelines-controller-5cf4d4867f-cw4kt   1/1     Running   0          51s
tekton-pipelines-webhook-57bfb4b4d7-bj85t      1/1     Running   0          49s
```

### TEKTON Dashboard Install

```kubectlコマンド
kubectl apply --filename https://storage.googleapis.com/tekton-releases/dashboard/latest/tekton-dashboard-release.yaml
```

```コマンド結果
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

ServiceのタイプをLoadBalancerに変更して、外部からアクセスできるようにします。

```kubectlコマンド
kubectl patch service tekton-dashboard -n tekton-pipelines -p '{"spec": {"type": "LoadBalancer"}}'
```

```コマンド結果
service/tekton-dashboard patched
```

EXTERNAL-IPを確認します。ここでは、コマンド結果から「140.83.50.157:9097」にブラウザでアクセスします。

```kubectlコマンド
kubectl get svc -n tekton-pipelines
```

```コマンド結果
NAME                          TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)                              AGE
tekton-dashboard              LoadBalancer   10.96.211.23    140.83.50.157   9097:30259/TCP                       109s
tekton-pipelines-controller   ClusterIP      10.96.111.154   <none>          9090/TCP,8080/TCP                    3m46s
tekton-pipelines-webhook      ClusterIP      10.96.124.31    <none>          9090/TCP,8008/TCP,443/TCP,8080/TCP   3m45s
```

### TEKTON CLI Install

```linuxコマンド
curl -LO https://github.com/tektoncd/cli/releases/download/v0.18.0/tkn_0.18.0_Linux_x86_64.tar.gz
```

OCI Cloud Shellの場合は、以下の手順を実施します。それ以外は、任意の場所に格納して、パスを通してください。

```linuxコマンド
cd
```

```linuxコマンド
mkdir bin
```

```linuxコマンド
tar -xvzf tkn_0.18.0_Linux_x86_64.tar.gz
```

```コマンド結果
LICENSE
README.md
tkn
```

```linuxコマンド
mv tkn bin
```

パスを通します。$homeはご自身のホームディレクトリになります。

```linuxコマンド
export PATH="$PATH:/home/$home/bin"
```

不要ファイルを削除します。

```linuxコマンド
rm -rf LICENSE README.md tkn_0.18.0_Linux_x86_64.tar.gz
```

tknコマンドを実行してバージョンを確認します。

```tknコマンド
tkn version
```

```コマンド結果
Client version: 0.18.0
Pipeline version: v0.24.1
Dashboard version: v0.17.0
```

### TEKTONパイプライン構築

本リポジトリにある'tekton'ディレクトリ内にある01～07のマニフェストファイルを'kubectl apply'コマンドを実行します。

```linuxコマンド
cd tekton
```

'annotations'のULR、'username'、'password'は、ご自身のコンテナレジストリの情報を入力してください。OCIRの場合は、各リージョンは[こちら](https://docs.oracle.com/ja-jp/iaas/Content/Registry/Concepts/registryprerequisites.htm)で確認してください。


```linuxコマンド
cat 01-secret-serviceaccount.yaml
```

```コマンド結果
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

```kubectlコマンド
kubectl apply -f 01-secret-serviceaccount.yaml
```

```コマンド結果
secret/basic-user-pass created
serviceaccount/build-bot created
```

'value'には、'git clone'するご自身のgitリポジトリを指定します。

```linuxコマンド
cat 02-gitPipelineResource.yaml
```

```コマンド結果
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

```kubectlコマンド
kubectl apply -f 02-gitPipelineResource.yaml
```

```コマンド結果
pipelineresource.tekton.dev/hello-world-go-git created
```

'value'には、ご自身のイメージリポジトリのパスを指定します。

```linuxコマンド
cat 03-imagePipelineResource.yaml
```

```コマンド結果
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

```kubectlコマンド
kubectl apply -f 03-imagePipelineResource.yaml
```

```コマンド結果
pipelineresource.tekton.dev/hello-world-go-image created
```

```kubectlコマンド
kubectl apply -f 04-task.yaml
```

```コマンド結果
task.tekton.dev/build-and-push created
```

```kubectlコマンド
kubectl apply -f 05-taskrun.yaml
```

```コマンド結果
taskrun.tekton.dev/build-push-task-run created
```

```kubectlコマンド
kubectl apply -f 06-pipeline.yaml
```

```コマンド結果
pipeline.tekton.dev/build-push-pipeline created
```

```kubectlコマンド
kubectl apply -f 07-pipelinerun.yaml
```

```コマンド結果
pipelinerun.tekton.dev/build-push-pr created
```

TEKTON Dashboardで状況を確認できます。以下コマンドの実行でも確認できます。

```tknコマンド
tkn pipelinerun logs build-push-pr
```

```コマンド結果
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