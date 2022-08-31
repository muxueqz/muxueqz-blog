Title: k8s管理利器
Date: 2022-08-31 09:20
Tags: kubernetes, DevOps
Category: 系统运维
Slug: k8s-tools
Author: muxueqz

# 帮助我们更高效的管理k8s

* kubectl命令补全
* kubenav
* kubernetes dashboard
* k9s

---
# kubenav
安装：
```bash
podman run --rm -it -p 14122:14122 \
  -v ~/.kube/config:/dev/shm/kube_config:ro \
  kubenav/kubenav:master \
  --kubeconfig=/dev/shm/kube_config --debug
```

---
# kubernetes dashboard
kubernetes官方出品，比较朴实，不能切换context

安装：
```bash
podman run --rm -it -p 14122:9090 \
  -v ~/.kube/config:/dev/shm/kube_config:ro \
  kubernetesui/dashboard:v2.6.1 \
  --kubeconfig /dev/shm/kube_config
```

---
# kubectl命令补全
```bash
kubectl completion bash > kubectl.bash
source kubectl.bash
```

---
# k9s
* 多个context切换
 * 切换qa和生产环境的k8s集群
* 管理deployments
* 查看日志
  * 查看上一次日志
* 管理pod
* 进入shell
* 管理ingress
