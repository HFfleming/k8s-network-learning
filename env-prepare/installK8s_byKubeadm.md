### 一：环境准备

1. 准备2台ubuntu虚拟机,地址分别为

   `192.168.186.128 bpf1`

   `192.168.186.129 bpf2`

   ![image-20230416215933269](D:\K8S\k8s-network\GitRepo\k8s-network-learning\env-prepare\assets\image-20230416215933269.png) 

   ubuntu 20.04 ，内核版本为5.13，ISO镜像下载链接为：

   链接：https://pan.baidu.com/s/18KqqHPwNveTk1_QM636SJA   提取码：5233
   

2. 给节点安装必要的工具

   ```shell
   apt install -y net-tools tcpdump  chrony bridge-utils tree wget iftop ethtool curl
   ```

   ![image-20230416174222893](D:\K8S\k8s-network\GitRepo\k8s-network-learning\env-prepare\assets\image-20230416174222893.png) 

3. 关闭swap分区

   ```bash
   sed -ri 's/.*swap.*/#&/' /etc/fstab
   swapoff -a
   ```

   ![image-20230416174416968](D:\K8S\k8s-network\GitRepo\k8s-network-learning\env-prepare\assets\image-20230416174416968.png) 

4. 修改内核配置

   ```shell
   # vi /etc/sysctl.conf
   net.ipv4.ip_forward = 1
   net.bridge.bridge-nf-call-ip6tables = 1
   net.bridge.bridge-nf-call-iptables = 1
   net.bridge.bridge-nf-call-arptables = 1
   ```

5. 添加阿里的repo镜像仓库源，**注意分开执行，不要全部一次性执行**

   ```shell
   apt-get update && apt-get install -y apt-transport-https
   apt-get install -y apt-transport-https
   apt upgrade -y
   curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
   tee /etc/apt/sources.list.d/kubernetes.list <<-'EOF'
   deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
   EOF
   apt-get update
   ```

   

6. 安装docker(版本为23+),注意分开执行，不要全部一次性执行

   ```shell
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
   sudo apt update
   sudo apt install docker-ce docker-ce-cli containerd.io
   mkdir -p /etc/docker
   cat <<EOF > /etc/docker/daemon.json
   {
     "registry-mirrors": ["https://pz0rz98b.mirror.aliyuncs.com"]
   }
   EOF
   systemctl daemon-reload
   systemctl restart docker
   systemctl enable docker
   ```

   ![image-20230416180747971](D:\K8S\k8s-network\GitRepo\k8s-network-learning\env-prepare\assets\image-20230416180747971.png) 

7. 由于我的两台机器 hostname 一致，为了避免后续安装k8s引起不必要的麻烦，修改两个机器的hostname，并在master节点 的/etc/host 中配置域名信息

   ```shell
   192.168.186.128 bpf1
   192.168.186.129 bpf2
   ```

8. 安装三件套 kubelet kubectl kubeadm

   ```bash
   apt-get install -y kubelet=1.23.4-00 kubeadm=1.23.4-00 kubectl=1.23.4-00 --allow-unauthenticated
   netplan apply 
   systemctl enable kubelet && systemctl restart kubelet
   ```

9. 配置kubelet 的cgroup 驱动

   ```bash
   # vi /etc/docker/daemon.json 
   {
     "exec-opts": ["native.cgroupdriver=systemd"],
     "registry-mirrors": ["https://pz0rz98b.mirror.aliyuncs.com"]
   }
   systemctl daemon-reload
   systemctl restart docker
   systemctl enable docker
   ```

10. master 节点kubelet 配置

    ```shell
    mkdir -p /var/lib/kubelet/
    cat > /var/lib/kubelet/config.yaml <<EOF      
    apiVersion: kubelet.config.k8s.io/v1beta1
    kind: KubeletConfiguration
    cgroupDriver: systemd
    EOF
    systemctl daemon-reload
    systemctl restart docker
    systemctl enable kubelet && systemctl restart kubelet
    systemctl daemon-reload
    systemctl restart docker
    systemctl enable docker
    ```

    校验检查

    ```shell
     # 执行命令 docker info|grep "Cgroup Driver"，回显systemd
     docker info|grep "Cgroup Driver"
     Cgroup Driver: systemd
    ```

11. 初始化K8s集群，只需在master 节点执行

    ```bash
    kubeadm config images pull --image-repository=registry.aliyuncs.com/google_containers
    #以下命令 设置未安装kube-proxy，为cilium提供安装环境，如果使用flannel 则会报错
    kubeadm init --kubernetes-version=v1.23.5 --image-repository registry.aliyuncs.com/google_containers --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12 --skip-phases=addon/kube-proxy --ignore-preflight-errors=Swap
    #若不使用cilium。则使用以下命令 
    kubeadm init --kubernetes-version=v1.23.5 --image-repository registry.aliyuncs.com/google_containers --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12 --ignore-preflight-errors=Swap
    ```

12. node节点加入到k8s集群

    ```bash
    kubeadm config images pull --image-repository=registry.aliyuncs.com/google_containers
    kubeadm join 192.168.186.128:6443 --token xxxxxx \
            --discovery-token-ca-cert-hash xxxxxxxx
    # kubeadm join 192.168.186.128:6443 --token jxewwj.faqhnmwnj1usnwo1 \
            #--discovery-token-ca-cert-hash sha256:24c4732baec2c5c6d08a383d35169302ee68ddd378407f73bcd027202b2d6763
    
    ```

13. 安装cni 组件

    ```bash
    kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
    kubectl taint nodes --all node-role.kubernetes.io/master-
    ```

14. 配置kubectl 操作集群

    ```shell
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```

    

