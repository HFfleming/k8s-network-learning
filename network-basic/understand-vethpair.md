### 一:veth pair 概念

![image-20230421105026706](./assets/image-20230421105026706.png) 



### 二： 通过containerLab构建网络topo

1. containerlab拓扑配置如下

   ```shell
   #1-setup-clab.sh
   #!/bin/bash
   set -v
   cat <<EOF>clab.yaml | clab deploy -t clab.yaml -
   name: veth
   topology:
     nodes:
       server1:
         kind: linux
         image: burlyluo/nettool
         exec:
         - ip addr add 10.1.5.10/24 dev net0
         
       server2:
         kind: linux
         image: burlyluo/nettool
         exec:
         - ip addr add 10.1.5.11/24 dev net0
         
       links:
         - endpoints: ["server1:net0","server2:net0"]
   EOF
   ```
   
1. 构建完成

   ![image-20230421105345075](./assets/image-20230421105345075.png)
   
1. 查看server2容器的网卡信息

   `ip a`
   
   ![image-20230421105509424](./assets/image-20230421105509424.png)

4. 查看server2容器的net0 网卡的相关信息(确认对端网卡)

   `ethtool -S net0` 

   <img src="./assets/image-20230421110157013.png" alt="image-20230421110157013" style="zoom:33%;" /> 

   对端id是24号，本身id是23号

5. 查看server1 的网卡信息 和net0网卡的对端

   ![image-20230421110439393](./assets/image-20230421110439393.png) 

6. 确认这两个网卡形成了虚拟网卡对
