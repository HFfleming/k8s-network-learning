# k8s-network-learning
kubernetes容器网络学习，本文档将会记录从第一步搭建环境，到cilium，flannel，calico 等网络模型的学习

![image-20230802141655030](./assets/image-20230802141655030.png) 

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

1. 理解网络中报文的传输，以及抓包技巧。熟悉路由过程中源/目的IP MAC地址的变化

   https://github.com/HFfleming/k8s-network-learning/blob/main/network-basic/IPandMAC.md
   
   
   
1. 如何使用veth pair

   https://github.com/HFfleming/k8s-network-learning/blob/main/network-basic/understand-vethpair.md
   
   

3. 理解vxlan组网模式，并通过containerLab 搭建vxlan网络拓扑加深理解

   https://github.com/HFfleming/k8s-network-learning/blob/main/network-basic/setup-vxlan-env-byContainerLab.md
   
   

4. BGP相关知识介绍

   https://github.com/HFfleming/k8s-network-learning/blob/main/network-basic/About-BGP.md

---

### 三：cilium 篇

1. Cilium 介绍，以及cilium安装的三种方式。（必须掌握）

    https://github.com/HFfleming/k8s-network-learning/blob/main/cilium-cni/how-to-install-cilium.md

2. Cilium Native Routing with kube-proxy 的搭建以及工作模式

     https://github.com/HFfleming/k8s-network-learning/blob/main/cilium-cni/Native-Routing-with-kubeProxy.md 

3. Cilium Native Routing with eBPF Host Routing 工作模式介绍

      https://github.com/HFfleming/k8s-network-learning/blob/main/cilium-cni/Native-Routing-with-eBPF-hostRouting.md

4. Cilium VxLan with eBPF Host Routing 工作模式分析

   https://github.com/HFfleming/k8s-network-learning/blob/main/cilium-cni/Cilium-VxLAN-with-eBPF-hostRouting.md

5. Cilium IPSec 环境搭建以及工作模式分析

   https://github.com/HFfleming/k8s-network-learning/blob/main/cilium-cni/IPSec/Cilium-IPSec-DataPath.md

6. Cilium WireGuard 环境搭建及工作模式分析

   https://github.com/HFfleming/k8s-network-learning/blob/main/cilium-cni/WireGuard/Cilium-WireGuard-DataPath.md

7. Cilium Socket LB 环境搭建以及特性分析

   https://github.com/HFfleming/k8s-network-learning/blob/main/cilium-cni/socketLB/Cilium-SocketLB.md

8. Cilium DSR 环境搭建以及特性分析

   https://github.com/HFfleming/k8s-network-learning/blob/main/cilium-cni/DSR/Cilium-DSR.md

9. Cilium DualStack 环境搭建以及特性分析

   https://github.com/HFfleming/k8s-network-learning/blob/main/cilium-cni/DualStack/Cilium-DualStack.md

10. Cilium LB IPAM 环境搭建以及功能验证

      https://github.com/HFfleming/k8s-network-learning/blob/main/cilium-cni/LB-IPAM/Cilium-LB-IPAM.md

11. Cilium Ingress特性 环境搭建以及功能验证

       https://github.com/HFfleming/k8s-network-learning/blob/main/cilium-cni/Ingress/Cilium-Ingress.md

12. Cilium GatewayAPI特性 环境搭建以及功能验证

       https://github.com/HFfleming/k8s-network-learning/blob/main/cilium-cni/GatewayAPI/Cilium-GatewayAPI.md

13. Cilium BGP ControlPlane特性介绍以及环境搭建和功能验证

       https://github.com/HFfleming/k8s-network-learning/blob/main/cilium-cni/BGP/Cilium-BGP-ControlPlane.md
