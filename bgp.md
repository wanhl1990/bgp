title: BGP 基础篇
speaker: 万海龙
transition: slide3
files: /js/demo.js,/css/demo.css,/js/zoom.js
theme: moon
usemathjax: yes

[slide]

# BGP协议学习 #

[slide data-transition="zoomin"]

## BGP的来历
------

EGP 外部网关协议有以下几个缺点：

* 没有发现路由环路的能力 {:&.rollIn}
* 不支持复杂的基于策略的路由 
* 不能充分地与IGP互相合作 
* 公布网络变化相当慢 

[slide data-transition="glue"]

## BGP的来历
------

既然EGP有这些缺点，那么BGP相对于它有哪些增强呢？ {:&.flexbox.vleft}

1. BGP引入了AS_PATH属性。用以判断收到路由的AS号是否为自己. {:&.moveIn}
2. 具备一套路由优选和路由控制策略。
3. BGP使用过程中，有个同步的概念。就是IGP必须与BGP路由同步。
4. 具备触发更新特征。


[slide data-transition="zoomout"]]

## 下面由几个方面的问题来引申出BGP的优点以及工作过程

* 单台设备需要收集和存储哪些信息？ {:&.rollIn}
* 如何交互和通信？
* 如何决策出最佳通信通道？

[slide data-transition="slide2"]
## 协议需要发布哪些信息？

最基本的是IP前缀和掩码、下一跳，这是一条路由的最简描述，任何一种路由协议都需要收集这些信息。为了支持路由优选，需要考虑路由的优先级（至少一种度量），并记录路由的来源（哪个AS发布，什么方式引入），还有优先级等（LOCAL_PREF，MED等）。 这些信息是在交互信息的update阶段来发出的。

[slide data-transttion="cards"]

##BGP的属性

分为4类：

* **公认强制（必遵）**
* **公认自决（自选）**
* **公认转发**
* **公认非转发**

[slide]

## 公认强制属性

***ORIGIN***：标记路由信息的来源。分为IGP，BGP，INCOMPLETE{:&.flexbox.vleft} 

***AS_PATH***：　包含在Update报文中路由信息锁经过的AS的序列。一般还分为AS_SET和AS_SEQUENCE两种 

***NEXT_HOP***: 定义到达目的地址的吓一跳地址。

[slide]

## 公认自决属性

***LOCAL_PREF***：本地优先级。告诉AS，系统内的路由器有多条路径时，本地优先级越高的，路由优先级就越高。一般仅传递给IBGP邻居。{:&.flexbox.vleft}

***ATOMIC_AGGREGATE***（原子聚合）：原子聚合属性指出已被丢失了的信息。

[slide]

## 公认转发

***AGGREGATOR***（聚合者）：次属性表明了实施路由聚合的BGP路由器ID和聚合路由的路由器AS号。{:&.flexbox.vleft}

***COMMUNITY***（团体）：次属性指功效一个公共属性的一组路由器。	

[slide]

## 公认非转发

***MED***（MULTI_EXIT_DISC，多出口区分）：通知AS以外的路由器采用哪条路到达AS。 MED值越低，优先级越高。{:&.flexbox.vleft}

***ORIGINATOR_ID***（起源ID）：路由器反射器会附加到这个属性。携带本AS路由器的ID，防止环路。

***CLUSTER_LIST***（簇列表）： 显示了采用的反射路径。

[slide]

## 如何存储这些路由信息？

BGP 存储路由信息的数据库交RIB（Routing Information Base）. 这个数据库还分为三个部分：

	1. Adj-RIBs-In，保存BGP Speaker从邻居学到的路由信息，即初始路由
	2. Loc-RIB，保存经过决策从Adj-RIBs-In选取的路由信息，即最优路由
	3. Adj-RIBs-Out，保存BGP Speaker发给邻居的路由信息，即发布路由

[slide]

#如何交互和通信？

[slide]
## 交互通信

BGP采用TCP 作为承载协议， 使用端口 179. 信息交互时，在TCP之上，增加BGP消息格式头。{:&flexbox.vleft}

[magic]
![](/bgp_head.png)
[/magic]

[slide data-transition="zoomout"]
## 报文头格式

 * Marker(16字节)：用于BGP认证，不使用认证时所有比特为1。
 * length(长度)：2字节　type(类型):1字节
 * 类型1：open协商BGP参数，建立邻居关系
 * 类型2：update传播BGP路由
 * 类型3：notification报告错误，中止邻居关系
 * 类型4：keepalive维持邻居关系，默认周期为60s

BGP合理的长度为19-4096。由于BGP在报文格式中普遍采用了TLV（Type，Length，Value）的形式，而不是固定长度固定字段的形式，使得BGP具有非常好的扩展性，在后期追加新类型支持新业务时，只需要定义新的类型编码和值，报文不需要做任何更改。{:&.flexbox.vleft} 

[slide]

## BGP消息类型--Open

**Open**： Open消息是BGP协议在TCP建立完成后的第一个消息，用于建立BGP对等体之间的连接关系。消息格式如图

![](/bgp_open.png)

[slide data-transition="horizontal3d"]

## BGP消息类型--Open

* version（版本）：1字节，现在为版本4
* my as (自己的AS号)：2字节， 写上自己的AS号 1-65535
* hold time（超时）：2字节。Keepalive 周期一般为HT的1/3。HT最小值为3秒，默认为180s。
* BGP　Identiffer（标识）：4字节，为loopback中最大的IP。
* optional parameters: 公布一些可选功能的支持。

[slide data-transition="stick"]

	Open 消息格式实例
<br><br>

![](/bgp_open_eg.png)

[slide data-transition="slide2"]	
## BGP消息类型--Update
	
**Update**：<span class="red" 用于对等体之间的路由信息交换。可以发布可达路由消息，也可以撤销不可达路由信息。<span> {:.highlight}

[slide]

## BGP消息类型--Update

**update消息格式**

![](/img/bgp_update.png)

[slide]
## BGP消息类型--Update

* unfeasible routes length（2字节）:不可到（无效）路由长度，如没有则为0
* total path attribute length（2字节）:BGP属性长度
* path attributes（可变长）：BGP路径属性
* nework layer reachability information(可变长):可达路由信息，里面包含前缀长度和网络号

[slide]

## *路径属性（path attributes）*

属性类型是2字节字段包括了属性标志字节和属性类型码字节。<br>

- 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 {:&.flexbox.vleft}
- +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
- |  Attr. Flags  |Attr. Type Code|
- +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

[slide]
## 属性标志

* 属性标志字节第一高位比特（Bit0）是可选比特。定义了属性是否是可选的（设为1）或者是公认的（设为0）。
* 属性标志字节第二高位比特（Bit1）是转发比特。定义一个可选的属性是否是转发的（如果设置为1）或者不是转发的（设为0）。公认属性的转发位必须设为1）。
* 属性标志字节的第三比特（Bit2）是部分比特。它定义了包括在可选转发属性内的信息是部分的（设置为1）还是完整的（设置为0）。（公认属性和可选非转发属性的部分位必须是0）。
* 属性标志字节的第四比特（Bit3）是扩展长度比特。定义了属性长度是1字节（如果设置为0）还是2字节（如果设置为1）。仅仅当属性值超过255字节的时候，扩展长度可以使用。
* 属性标志字节低字节顺序4比特没有被使用。必须填0（接收时不处理）。

[slide]

属性类型抓包实例：

![](/img/bgp_update_attri.png)

[slide]

##属性码 -- ORIGIN

Type code :**1**

**公认强制属性** 

- 0         IGP – 网络层可达信息和来源AS同内部
- 1         EGP – 网络层可达信息通过EGP学习
- 2         INCOMPLETE –网络层可达信息通过别的方式学习

[slide]


##属性码 -- AS_PATH

Type code :**2**

**公认必遵属性** 

由一系列AS路径段组成。每一个AS路径段表示为TVL<type，length，value>。<br>

**Type**{:&.flexbox.vleft}

	1. AS_SET: 在UPDATE消息中的路由经过的AS的无序集
	2. AS_SEQUENCE: 在UPDATE消息中的路由经过的AS的有序集

**Length** 1字节长度的字段， 包含了在路径段值字段的AS的数量{:&.flexbox.vleft}

**Value** 路径段值字段包含了一个或者多个AS号，每一个编码为2字节长度的字段。{:&.flexbox.vleft}

[slide]


##属性码 -- NEXT_HOP

Type code :**3**

**公认必遵属性** 

这是一个公认强制属性，它定义了作为到达目的地作为下一跳的边界路由器的IP地址，目的地列表于UPDATE消息的网络层可达字段。{:&.flexbox.vleft}

[slide]

##属性码 -- MULTI_EXIT_DISC

Type code :**4**

**可选非转发属性** 

4字节非负整数。属性值可以被BGP发言者决策过程在相邻自治系统中区分多个出口。 {:&.flexbox.vleft}

[slide]

##属性码 -- LOCAL_PREF

Type code :**5**

**公认自决属性** 

4字节非负整数。BGP发言者使用它通知别的BGP在自己的自治系统中源发言者通告路由的优先程度。用于IBGP中，不转发给EBGP。{:&.flexbox.vleft}

[slide]

##属性码 -- ATOMIC_AGGREGATE

Type code :**6**

**公认自决属性** 

如果BGP发言者收到两条重叠的路由，其中一条是另一条的子集。在对外发布的时候，如果它选择了更粗略的哪条路由时，传递消息时需携带 ***ATOMIC_AGGREGATE*** 属性，以告知邻居。它实际上是一种警告信息。{:&.flexbox.vleft}

[slide]


##属性码 -- AGGREGATOR

**Type code :7**

**可选转发属性** 

它是ATOMIC_AGGREGATE属性的补充。如上一张所述，ATOMIC_AGGREGATE是路有消息丢失的警告，那么***AGGREGATOR*** 属性补充了路由信息在哪丢失了。 它保护了路由聚合的AS号码和形成聚合路由的BGP发言者的IP地址{:&.flexbox.vleft}

[slide]

#团体属性

* 8   COMMUNITY 可选转发属性。 它是一组共享相同属性的目的地集合。
* 9   ORIGINATOR_ID 可选非转发属性。 用于标识路由反射器。
* 10  CLUSTER_ID 可选非转发属性。用于标识路由反射器组。

[slide]

## Network Layer Reachability Information 网络层可达信息（NLRI）

可达信息编码时作为一个或者多个二元组格式为〈长度，前缀〉，它们的字段描述如下：{:&.flexbox.vleft}


- +---------------------------+ {:&.flexbox.vleft}
- |   Length (长度，1字节 )  	  |
- +---------------------------+
- |   Prefix (变量)       	  |
- +---------------------------+

1. 长度：长度字段指示了IP地址前缀的比特长度。0指示一个匹配了所有IP地址的前缀（前缀本身0字节）
2. 前缀：前缀区字段包含了IP地址前缀并跟随足够的填充比特使该区字段的结尾能够落在字节边界

   <span class="yellow"> eg. 10.1.1.0/24, 192.168.0.0/16 </span>

[slide]
##NLRI 抓包实例

![](/img/bgp_nlri.png)

[slide]

## BGP消息类型--Update

当路由失效时， 发送格式：

* unfeasible routes length（2字节）:不可到路由长度，这路由有不可达路由
* withdrawn routes:撤消路由条目，包括前缀长度和网络号
* total path attribute length（2字节）:这里为0,没有数据
	
[slide data-transition="slide3"]	

路由失效时，抓包图
![](/img/bgp_update_unfeasible.png)

[slide]

## Update 消息总结

路由一个UPDATE消息可以仅仅撤销路由，这样就不需要包括路径属性或者网络层可达信息。相反，也可以仅仅通告可达路由，这样WITHDRAWN ROUTES不需要了。


[slide data-transition="slide1"]	
## BGP消息类型--Keepalive
	
**Keepalive**： 是一个类似探测、保活的消息。BGP会周期性的向对等体发送该消息，如果收到正确回复，那么就说明对等体还“活着”。长度为19，无实质data信息。周期为Hold time 的三分之一。最小为1s（建议采用默认值）。消息格式如图：{:&.flexbox.vleft}

![](/img/bgp_keepalive.png)

[slide]

## BGP消息类型-- Notification

**Notification**： 即当BGP检测到有错误状态时，向对等体发送该消息，通知本身有误，然后断开BGP连接。{:&.flexbox.vleft}

报文格式如图：

![](/img/bgp_notify.png)

[slide data-transition="cards"]

5.**BGP状态**

	1. IDLE： 初始状态，协议激活后开始初始化，复位计时器。发起第一个TCP连接，并开始listen 对等体发起的连接，同时转向Connect状态。
	2. connect：开始TCP连接，并等待TCP连接成功的消息。连接成功，进入OpenSent；连接失败，进入Active状态。
	3. Active：此时。在连接计时器超时之前，BGP总是尝试建立TCP连接。若计时器超时，则退回Connect状态，重新开始TCP连接；如果TCP连接成功，就转为OpenSent状态。
	4. Open Sent：TCP连接已建立，自己发送第一个Open报文，等待接收对等体Open报文。对收到的报文进行检查，发现错误则发送Notification消息报文并退回Idel状态；检查无误后发送Keepalive消息报文。Keepalive 计时器并开始即使，并转为confirmed状态。
	5. Open Confirm：BGP等待Keepalive报文，同时保持计时器。如果收到了Keepalive报文，就转为Established状态，邻居关系协商完毕。 如果收到Notification消息，BGP退回Idle 状态。
	6. Establish：即已经与对等体建立了邻居关系。路由器与对等体交换Update、Keepalive、Notification报文。如果收到正确的Update或Keepalive 报文， 就认为对端是正确运行，本地重置Hold timer计时器。如果收到Notification，本地转为Idle状态。如果收到错误的Update消息，本地发送Notification，并改本地状态为Idle状态。如果收到TCP拆链通知，本地关闭BGP连接。并回到Idle状态。

[slide]


6.**BGP路由通告原则**

	1. 多条路径时，BGP发言者只选最优的给自己使用。
	2. BGP发言者只把自己使用的路由（最优路由）通告给邻居。
	3. BGP发言者从EBGP获得的路由会想自己所有的BGP邻居通过（包括EBGP和IBGP）。
	4. BGP发言者从IBGP获得的路由不向自己的IBGP邻居通过（反射器除外）。
	5. BGP发言者IBGP获得路由是否通告给自己的EBGP邻居需要根据IGP和BGP同步的情况来决定。

[slide]	

7.**BGP属性**

	
	1. 公认必遵：
		
		ORIGIN：标记路由信息的来源。分为IGP，BGP，INCOMPLETE 

		AS_PATH：　包含在Update报文中路由信息锁经过的AS的序列。一般还分为AS_SET和AS_SEQUENCE两种 

		NEXT_HOP: 定义到达目的地址的吓一跳地址。
		
	2. 公认可选属性：
	
		LOCAL_PREF：本地优先级。告诉AS，系统内的路由器有多条路径时，本地优先级越高的，路由优先级就越高。

		ATOMIC_AGGREGATE（原子聚合）：原子聚合属性指出已被丢失了的信息。
	3. 可选过渡（转发）属性：
		
		AGGREGATOR（聚合者）：次属性表明了实施路由聚合的BGP路由器ID和聚合路由的路由器AS号。

		COMMUNITY（团体）：次属性指功效一个公共属性的一组路由器。	

	4. 可选非过度（转发）属性

		MED（MULTI_EXIT_DISC，多出口区分）：通知AS以外的路由器采用哪条路到达AS。 MED值越低，优先级越高。

		ORIGINATOR_ID（起源ID）：路由器反射器会附加到这个属性。携带本AS路由器的ID，防止环路。

		CLUSTER_LIST（簇列表）： 显示了采用的反射路径。


----------
[slide]
