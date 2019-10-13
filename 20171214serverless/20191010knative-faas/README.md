# 个人快速搭建serverless快速指南（建设中，目前只记录一些操作）
针对ubuntu18.04版本安装
```sh
sudo apt-get update
sudo apt-get install snapd -y
```

microk8s
```sh
sudo snap install --classic microk8s
sudo snap alias microk8s.kubectl kubectl
```
k8s.gcr.io被墙问题
/var/snap/microk8s/current/args/文件夹所有toml后缀的文件的k8s.gcr.io替换为mirrorgooglecontainers，然后重启containerd.service
sed -i 's/k8s.gcr.io/mirrorgooglecontainers/g' `grep -rl  --include="*.yaml" --include="*.toml"  "k8s.gcr.io"  /var/snap/microk8s/current/args/`
:%s/k8s.gcr.io/mirrorgooglecontainers/g
sudo systemctl restart snap.microk8s.daemon-containerd.service
安装个dashboard方便可视化看数据
```sh
sudo microk8s.status
sudo microk8s.enable dns dashboard rbac
sudo microk8s.kubectl -n kube-system edit configmap/coredns 
# 编辑 forward . 223.5.5.5 223.6.6.6，换阿里云dns，因为8.8.8.8是谷歌的会导致无法访问
# vim:%s/8.8.8.8\ 8.8.4.4/223.5.5.5\ 223.6.6.6/g
echo '#!/bin/bash
namespace=${1-"kube-system"}
for podname in $(sudo microk8s.kubectl -n $namespace get -o=name pod)
do
    sudo microk8s.kubectl  -n $namespace delete pod $(basename $podname)
done
for podname in $(sudo microk8s.kubectl -n $namespace get -o=name pod)
do
    mkdir -p $(dirname $podname)
    echo $(basename $podname)
    sudo microk8s.kubectl -n $namespace get pod  -o yaml $(basename $podname) > $podname.yaml
    sed -i "s/k8s.gcr.io/mirrorgooglecontainers/g" $podname.yaml
    sudo microk8s.kubectl  -n $namespace replace -f $podname.yaml
done' > replace_mirror.sh
sh replace_mirror.sh
sudo microk8s.kubectl cluster-info
cat /snap/microk8s/current/basic_auth.csv # 获取账号密码password,user,uid,"group1,group2,group3"
sudo microk8s.config # 获取登陆密码,注意最好拷贝出来看看不然容易拷贝错误
sudo kubectl get pods --all-namespaces
sudo microk8s.kubectl -n kube-system get pod # 看status，dashboard有没有正常启动
sudo microk8s.kubectl -n kube-system describe pod  # 看描述是否报错
sudo microk8s.kubectl -n kube-system get secret | grep default-token | cut -d " " -f1 # 获取token
sudo microk8s.kubectl -n kube-system describe secret default-token-djlf8 # default-token-djlf8是从上条命令获取的，每个人不一样
```
打开链接选择token登陆，使用上面显示的token
https://{你的IP}:16443/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/


查看是否安装成功
```sh
sudo microk8s.kubectl get nodes
sudo microk8s.kubectl get services
```
安装Knative
```sh
echo 'N;' | sudo microk8s.enable knative
```
查看是否安装完成
```sh
sudo kubectl get pods -n knative-serving
sudo kubectl get pods -n knative-eventing
sudo kubectl get pods -n knative-monitoring
```
// 开启rbac强行给匿名用户赋权超级管理员
  sudo kubectl create clusterrolebinding anonymous-cluster-admin \
  --clusterrole=cluster-admin \
  --group=system:anonymous

  sudo kubectl create clusterrolebinding unauthenticated-cluster-admin \
  --clusterrole=cluster-admin \
  --group=system:unauthenticated

  sudo kubectl delete clusterrolebinding anonymous-cluster-admin unauthenticated-cluster-admin