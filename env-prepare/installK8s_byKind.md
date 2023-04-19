### 一： kind 是什么

kind 是 Kubernetes in Docker 的简写，是一个使用 Docker 容器作为 Nodes，在本地创建和运行 Kubernetes 群集的工具。适用于在本机创建 Kubernetes 群集环境进行开发和测试。

<img src="./assets/kind-diagram.webp" alt="kind" style="zoom:33%;" /> 



### 二:  环境准备

1.  准备一台机器2u4g 即可

2.  机器上安装docker

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

3. 机器上安装kubectl

   ```bash
   curl -LO https://dl.k8s.io/release/1.23.4/bin/linux/amd64/kubectl
   sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
   kubectl version --client
   ```

4. 机器上安装golang运行环境

   下载go，这里选择go1.19版本

   ```bash
   wget https://golang.google.cn/dl/go1.19.2.linux-amd64.tar.gz
   tar -C /usr/local/ -xzf go1.19.2.linux-amd64.tar.gz
   ```

   修改/etc/profile文件

   ```bash
   #vim /etc/profile
   export GOROOT=/usr/local/go
   export PATH=$PATH:$GOROOT/bin
   ```

   修改完成后

   ```bash
   source /etc/profile
   ```



### 三： 安装 kind & kind 创建k8s集群

1.  源编译安装

   ```bash
   go install sigs.k8s.io/kind@v0.18.0
   ```

   如果安装报错，报错信息为超时之类的，可能需要修改goproxy

   ```bash
   go env -w GOPROXY="https://goproxy.cn,direct"
   ```

2. kind 安装完成后，直接使用kind 命令会报错：`kind: command not found`

   这是因为kind安装完成后，被放在了 `$(go env GOPATH)/bin` 下

   `export PATH=$PATH:/root/go/bin/`  解决

   

3.  脚本化 安装kind： `sh 1-setup-env.sh`

   ```shell
   #1-setup-env.sh
   #! /bin/bash
   date
   set -v
   
   # 1.prep nocNI env
   cat <<EOF |kind create cluster --name=flannel-udp --image=kindest/node:v1.23.4  --config=-
   kind: Cluster
   apiVersion: kind.x-k8s.io/v1alpha4
   networking:
     disableDefaultCNI: true  #kind 默认使用rancher cni，我们不需要该cni
     podSubnet: "10.244.0.0/16"
   nodes:
     - role: control-plane
     - role: worker
     - role: worker
   EOF
   
   # 2. remove taints
   controller_node=`kubectl get nodes --no-headers -o custom-columns=NAME:.metadata.name |grep control-plane`
   kubectl taint nodes $controller_node node-role.kubernetes.io/master:NoSchedule-
   kubectl get nodes -owide
   
   # 3. install CNI
   kubectl apply -f ./flannel.yaml
   #kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
   
   #4. install necessary tools
   for i in $(docker ps -a --format "table {{.Names}}" |grep flannel-udp)
   do
                   echo $i
                   #docker cp ./bridge $i:/opt/cni/bin/
                   docker cp /usr/bin/ping $i:/usr/bin/ping
                   docker exec -it $i bash -c "apt-get -y update > /dev/null && apt-get -y install net-tools tcpdump lrzsz > /dev/null 2>&1"
   done
   
   ```
   
4. 安装过程如下所示

   ![image-20230419000102300](./assets/image-20230419000102300.png) 

   

5. 安装完成后，可以执行命令 `kind get clusters`  `kubectl get xxx`查看安装的集群

   ![image-20230419000228157](./assets/image-20230419000228157.png) 



6. 如果安装完集群，`kubectl get po -A` ,pod启动失败

   ![image-20230419173739154](./assets/image-20230419173739154.png) 

   查看异常状态pod日志，发现cni插件相关报错，缺少bridge

   `failed to delegate add: failed to find plugin "bridge" in path [/opt/cni/bin]`

   节点上没有相关cni组件，所以我们可以下载相关plugin，然后docker cp到容器节点内

   ```shell
   wget https://github.com/containernetworking/plugins/releases/download/v1.1.0/cni-plugins-linux-amd64-v1.2.0.tgz
   mkdir -p /opt/cni/bin
   tar -C /opt/cni/bin -xzf cni-plugins-linux-amd64-v1.2.0.tgz
   for i in $(docker ps -a --format "table {{.Names}}" |grep flannel-udp);do echo $i;docker cp /opt/cni/bin/bridge $i:/opt/cni/bin/;done
   ```

   然后查看集群信息,一切正常

   ![image-20230419174325244](./assets/image-20230419174325244.png) 

