
一、开启路由器
```
<Huawei>system-view

[Huawei]sysname r1  //修改设备名称
```
 
二、查看配置命令
```
[r1]display current-configuration  //查看当前生效的配置

[r1]display this  //查看当前位置的配置
```
 
三、配置DHCP
```
[r3]dhcp enable  //打开DHCP服务

[r3]ip pool qw   //创建名为qw的地址池塘

Info: It's successful to create an IP address pool.    //系统提示

[r3-ip-pool-qw]network 192.168.1.1 mask 255.255.255.240  //定义池塘地址范围

[r3-ip-pool-qw]gateway-list 192.168.1.2  //定义网关

[r3-ip-pool-qw]dns-list 114.114.114.114  //定义DNS服务

[r3-GigabitEthernet0/0/2]dhcp select global   //接口处打开调用
```
 
四、静态路由
```
[r1]ip route-static 172.16.3.0 24 172.16.2.2   //手动写静态路由

                    目标网络号  掩码  下一跳

[r1]ip route-static 0.0.0.0 0 172.16.2.2    //添加缺省路由

[r1]ip route-static 10.1.0.0 22 NULL 0    //配置空接口路由

[r1]display ip routing-table    //查看路由器的路由表

[r1]display ip routing-table protocol ospf    //查看具体协议的路由表   

[r1]interface LoopBack 0    /// 创建环回接口

```
五、动态路由

5.1RIP

1、RIP基础配置
```
[r1]rip 1    //启动协议时必须配置进程号，仅具有本地意义

[r1-rip-1]version 1   //选择版本，以版本1为例，版本2同版本1

[r1-rip-1]network 192.168.1.0    //宣告路由，基于主类范围宣告

[r1-rip-1]undo summary   //关闭自动汇总

 

[r1]display ip routing-table protocol rip   //查看RIP路由表
```
2、RIP扩展配置

1）手工汇总
```
[r1]interface GigabitEthernet 0/0/1   //进入更新发出的接口

[r1-GigabitEthernet0/0/1]rip summary-address 10.1.0.0 255.255.252.0
```
2）缺省路由--在连接运营商的边界路由器的协议中配置
```
[r3]rip 1   //边界路由器的RIP中

[r3-rip-1]default-route originate    //下发缺省
```
 

5.2 OSPF

1、OSPF基础配置
```
[r1]ospf 1 router-id 1.1.1.1   //启动协议，需要配置进程号，具有本地意义，可以选择配置RID，若不配置，则路由器自己选择

[r1-ospf-1]area 0   //进入区域

[r1-ospf-1-area-0.0.0.0]network 172.16.1.0 0.0.0.255  //宣告路由或接口

 

[r1]display ip routing-table protocol OSPF   //查看OSPF路由表

[r1]display  ospf peer  brief    //查看OSPF邻居表
```
2、OSPF扩展配置

1）手工汇总
```
[r1]ospf 1

[r1-ospf-1]area  1   明细路由所在区域

[r1-ospf-1-area-0.0.0.1]abr-summary 3.3.2.0 255.255.254.0
```
2）缺省路由--在连接运营商的边界路由器的协议中配置
```
[r4]ospf  1

[r4-ospf-1]default-route-advertise

 ```
六、acl访问控制列表
```
[r1]acl 2000  //创建基本ACL列表

[r1-acl-basic-2000]rule deny source 172.16.1.254     0    //添加规则，拒绝某ip

[r1-acl-basic-2000]rule deny source 172.16.1.0    0.0.0.255  //添加规则，拒绝一个范围，之后为反掩码

[r1-acl-basic-2000]rule deny source any  //添加规则，拒绝所有
```
所有的动作都可以换为permit，即允许

 
七、NAT网络地址转换

将私有地址转换为公有地址

1、静态NAT（一对一）：一个公网地址对应一个私网地址
```
[r2]nat static global 202.100.1.100 inside 172.16.1.10 netmask 255.255.255.0
```
2、动态NAT（多对多）：多个公网地址对应多个私网地址

1）定义多个公网地址
```
[r2]nat address-group 1  202.100.1.10     202.100.1.20   
```
2）定义多个私网地址
```
[r2]acl 2000

[r2-acl-basic-2000]rule permit source 172.16.1.0 0.0.0.255
```
3）配置NAT
```
[r2]interface GigabitEthernet 0/0/2   //公网接口

[r2-GigabitEthernet0/0/2]nat outbound 2000 address-group 1
```
3、Easy-IP：一对多，此时公网地址为边界路由器的出口地址

1）定义多个私网地址
```
[r2]acl 2000

[r2-acl-basic-2000]rule permit source 172.16.1.0 0.0.0.255
```
2）直接配置NAT
```
[r2]interface GigabitEthernet 0/0/2

[r2-GigabitEthernet0/0/2]nat outbound 2000  //直接用此接口的公网IP地址做转换
```
4、NAT服务器（端口映射）：将内网的服务器映射到互联网供互联网访问
```
[r2-GigabitEthernet0/0/2]nat server protocol tcp global 202.100.1.100 23 inside 172.16.1.100 23
```
 
八、VLAN

1、创建VLAN
```
[sw1]vlan 1    //创建单个VLAN

[sw1]vlan batch 30 40    //创建多个单个VLAN

[sw1]vlan batch 40 to 50  //创建多个连续的VLAN
```
2、将交换机的接口划分VLAN

方法1：

单个接口划分
```
[sw1]interface GigabitEthernet 0/0/1                    //进入接口

[sw1-GigabitEthernet0/0/1]port link-type access   //修改接口类型

[sw1-GigabitEthernet0/0/1]port default vlan 10     //将接口划入某VLAN

多个接口同时划分

[sw1]interface range GigabitEthernet 0/0/1 to GigabitEthernet 0/0/10 //同时进入多个接口

[sw1-GigabitEthernet0/0/1]port link-type access //修改接口类型

[sw1-GigabitEthernet0/0/1]port default vlan 20     //将多个接口同时划入某VLAN
```
方法2：

单个接口划分
```
[sw1]interface GigabitEthernet 0/0/2                    //进入接口

[sw1-GigabitEthernet0/0/2]port link-type access   //修改接口类型

[sw1]vlan 10                                                          //进入VLAN

[sw1-vlan10]port GigabitEthernet 0/0/2                //将某接口划入此VLAN

多个接口同时划分

[sw1]interface range GigabitEthernet 0/0/1 to GigabitEthernet 0/0/10 //同时进入多个接口

[sw1-GigabitEthernet0/0/1]port link-type access //修改接口类型

[sw1-vlan10]port GigabitEthernet 0/0/2 to GigabitEthernet 0/0/10 //同时划入多个接口
```
3、TRUNK干道--用于SW---SW，当一个接口类型为TRUNK时，则此接口不属于任何一个VLAN，但是可以传递所有VLAN的数据

  默认不允许所有vlan通过，需要手工添加VLAN允许列表
```
[sw1]interface GigabitEthernet 0/0/5      //进入接口

[sw1-GigabitEthernet0/0/5]port link-type trunk    //修改接口类型为trunk类型

[sw1-port-group-link-type]port trunk allow-pass vlan 10 20   //添加允许列表
```
4、VLAN之间的通信--单臂路由或者三层交换机
```
[r1]interface GigabitEthernet 0/0/0.1   //创建子接口

[r1-GigabitEthernet0/0/0.1]dot1q termination vid 10   //本子接口可以作为VLAN10的网关

[r1-GigabitEthernet0/0/0.1]ip address 172.12.3.0  24  //配置ip地址

[r1-GigabitEthernet0/0/0.1]arp broadcast enable   //开启ARP广播功能

 ```

九、GRE 通用路由封装

VPN虚拟专用网络 --- 让两个网络穿越中间网络来直接通讯，逻辑的在两个网络间建立了一条新的点到点直连链路；
```
[r1]interface Tunnel 0/0/0    //创建虚拟接口

[r1-Tunnel0/0/0]ip address 10.1.1.1 24  //配置IP地址

[r1-Tunnel0/0/0]tunnel-protocol gre    //修改接口模式

[r1-Tunnel0/0/0]source 12.1.1.1     // 公有的源IP地址

[r1-Tunnel0/0/0]destination 23.1.1.2  //
```
 
十、MGRE

中心站点配置
```
[r1]interface Tunnel0/0/0    //创建tunnel口

[r1-Tunnel0/0/0]ip address 10.1.1.1 255.255.255.0   //配置接口ip地址

[r1-Tunnel0/0/0]tunnel-protocol gre p2mp  //先修改接口模式为多点GRE

[r1-Tunnel0/0/0]source 15.1.1.1  //再定义公有的源IP地址

[r1-Tunnel0/0/0]nhrp entry multicast dynamic   //本地成为NHRP中心，同时可以进行伪广播

[r1-Tunnel0/0/0]nhrp network-id 100   //默认为0号，该网段内所有节点tunnel接口必须为相同域

   伪广播—当目标IP地址为组播或广播地址时，将流量基于每个用户进行一次单播；外层报头为单播报头，内层报头为组播或广播报头；该功能不开启，正常基于组播和广播工作的动态路由协议将无法正常使用；

 

[r1]dis nhrp peer all  //查看分支站点注册结果
```
 

分支站点：
```
[r1]interface Tunnel0/0/0    //创建tunnel口

[r1-Tunnel0/0/0]ip address 10.1.1.2 255.255.255.0   //配置接口ip地址

[r1-Tunnel0/0/0]tunnel-protocol gre p2mp     //先修改接口模式为多点GRE

[r1-Tunnel0/0/0]source GigabitEthernet0/0/2  //假设分支站点ip地址不固定

[r1-Tunnel0/0/0]nhrp network-id 100       //再定义公有的源IP地址

[r1-Tunnel0/0/0]nhrp entry 10.1.1.1 15.1.1.1 register    //分支需要到中心站点注册
```
