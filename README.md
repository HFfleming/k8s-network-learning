# k8s-network-learning
kubernetes容器网络学习，本文档将会记录从第一步搭建环境，到cilium，flannel，calico 等网络模型的学习

![image-20230419104432029](./assets/image-20230419104432029.png)

---

### 一： 环境准备篇

1. 如何使用kubeadm 搭建kubernetes集群

   https://github.com/HFfleming/k8s-network-learning/blob/main/env-prepare/installK8s_byKubeadm.md

   

2. 如何使用kind 搭建kubernetes 集群

   https://github.com/HFfleming/k8s-network-learning/blob/main/env-prepare/installK8s_byKind.md

   

3. 如何搭建私人镜像仓库

   https://github.com/HFfleming/k8s-network-learning/blob/main/env-prepare/how-to-setup-private-repo.md
   
   

4. 如何使用containerLab构建网络拓扑

   https://github.com/HFfleming/k8s-network-learning/blob/main/env-prepare/install_containerLab.md
   
   

---

### 二： 网络基础篇

1. 如何使用veth pair

   https://github.com/HFfleming/k8s-network-learning/blob/main/network-basic/understand-vethpair.md
