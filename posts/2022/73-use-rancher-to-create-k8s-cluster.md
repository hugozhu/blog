---
date: 2022-08-16
layout: post
title: Use Rancher to manage k8s cluster
description: How to use Rancher to manage k8s cluster
categories:
- Blog
tags:
- Rancher K8s

---

{:toc}


# 必要的集群内部通讯端口要在防火墙上打开
2379, 2380, 80, 443, 22, 6443, 10250

# 清理干净 https://rancher.com/docs/rancher/v2.5/en/cluster-admin/cleaning-cluster-nodes/
```
docker rm -f $(docker ps -qa)
# docker rmi -f $(docker images -q)
docker volume rm $(docker volume ls -q)

for mount in $(mount | grep tmpfs | grep '/var/lib/kubelet' | awk '{ print $3 }') /var/lib/kubelet /var/lib/rancher; do sudo umount $mount; done

sudo rm -rf /etc/ceph \
       /etc/cni \
       /etc/kubernetes \
       /opt/cni \
       /opt/rke \
       /run/secrets/kubernetes.io \
       /run/calico \
       /run/flannel \
       /var/lib/calico \
       /var/lib/etcd \
       /var/lib/cni \
       /var/lib/kubelet \
       /var/lib/rancher/rke/log \
       /var/log/containers \
       /var/log/kube-audit \
       /var/log/pods \
       /var/run/calico


sudo ip link delete flannel.1

sudo reboot

sudo iptables -L -t nat
sudo iptables -L -t mangle
sudo iptables -L

```

# 启动管理节点
sudo docker run --privileged -d --restart=unless-stopped -p 80:80 -p 443:443 rancher/rancher

# 登录
获取初次安装密码：docker logs  $(docker ps -qa)  2>&1 | grep "Bootstrap Password:"
设置Server URL为内网地址：https://192.168.85.40，所有节点必须能访问这个服务器


# 安装节点
docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run  rancher/rancher-agent:v2.6.6 --server https://<api_server> --token <token> --ca-checksum <check_sum> --etcd --controlplane --worker



# 安装APISIX
```
helm repo add apisix https://charts.apiseven.com
helm repo update
helm install apisix apisix/apisix --create-namespace  --namespace apisix

```

# 安装ARGOCD
