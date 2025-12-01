# k8s 常用用法
## 一些概念：
- cluster: k8s集群
- node: 集群中一台机器. 一台安装k8s的物理机器
- pod: 最小的调度单位，运行容器的地方
- deploy: 管理pod的控制器
- service: 暴露pod的访问方式
- configmap: 配置信息
- secret:存放密码
- namespace: 命名空间

## 常用命令：
1.基本命令
```bash
kubectl version       # 查看 kubectl 和 k8s 版本
kubectl cluster-info  # 查看集群信息
kubectl get nodes     # 查看所有节点
kubectl describe node <node-name>  # 查看节点详细信息
```

2.查看资源列表
```bash
kubectl get pods # 查看所有pods
kubectl get svc # 查看所有的svc
kubectl get deployments # 查看所有deploy
kubectl get rs
kubectl get configmap
kubectl get secret
kubectl get ingress
kubectl get pvc
kubectl get pv

kubectl get all -n <namespace> # 查看指定命名空间的资源
kubectl get ns # 查看所有命名空间
```

3.查看资源详情
```bash
kubectl describe pod <pod-name>
kubectl describe deployment <deployment-name>
kubectl describe svc <service-name>

kubectl get pod <pod-name> -o yaml # 查看某个资源的yaml文件
```

4.应用yaml配置
```bash
kubectl apply -f <file.yaml>        # 创建或更新
kubectl create -f <file.yaml>       # 只创建

kubectl create namespace dev
kubectl create deployment myapp --image=nginx
kubectl expose deployment myapp --port=80 --type=NodePort
```

5.更新资源
```bash
kubectl scale deployment myapp --replicas=5 # 扩容缩容

kubectl rollout restart deploy xxx # 滚动重启更新

kubectl rollout restart deploy -n myns # 重启所有

kubectl rollout restart daemonset <name> #重启 DaemonSet / StatefulSet
kubectl rollout restart statefulset <name>

# 回滚
kubectl rollout status deployment/myapp
kubectl rollout undo deployment/myapp

```

6.删除资源
```bash
kubectl delete pod <pod-name>
kubectl delete deployment <deployment-name>
kubectl delete svc <service-name>
kubectl delete ns <namespace>
kubectl delete -f <file.yaml>

```

7.查看日志

```bash
kubectl logs <pod-name> #查看某个pod的日志
kubectl logs <pod-name> -c <container-name>  # Pod 多容器
kubectl logs -f <pod-name>  # 实时追踪日志

# 进入容器
kubectl exec -it <pod-name> -- /bin/bash
kubectl exec -it <pod-name> -c <container-name> -- sh
```

8.命名空间相关
```bash
kubectl get pods -n dev
kubectl describe pod <pod> -n dev
kubectl apply -f xxx.yaml -n dev

#设置默认namespace
kubectl config set-context --current --namespace=dev

```

9.资源状态检查
```bash
kubectl top node     # 查看节点 CPU/内存（需 metrics-server）
kubectl top pod      # 查看 Pod CPU/内存
kubectl get events   # 查看集群事件（排障很有用）

```