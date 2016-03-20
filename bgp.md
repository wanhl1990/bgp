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

## 1.**BGP 由来**

>路由协议分为两种，内部网关协议和外部网关协议。最开始的外部网关协议为EGP。 但是由于当初设计的比较简单，漏洞与问题比较多，所以就有了BGP的设计与推广。 相对与EGP来讲， BGP具备以下有点 **解决了环路问题、收敛问题、触发问题** 等。

下面是引自维基百科的相关说明：
>BGP的邻居关系（或称通信对端/对等实体）是通过人工配置实现的，对等实体之间通过TCP(端口179)会话交互数据。BGP路由器会周期地发送19字节的保持存活keep-alive消息来维护连接（默认周期为30秒）。在路由协议中，只有BGP使用TCP作为传输层协议。

[slide data-transition="slide2"]]
## 2.**BGP 的一些基本知识**
	
- BGP采用了TCP作为传输层协议，端口179， 可以利用TCP的***可靠性传输机制、重传、排序***等机制来确保协议报文交互的**可靠性**

- 在两个AS之间建立BGP的话，由于信任的原因， BGP不通过自动发现，而是使用手动配置的方式，使两端建立其BGP连接

[slide]

## 3.**BGP 的一些特点**

	1. 实现自治系统间通信，传播网络的可达信息。（NLRI） {:&.fadeIn}
	2. 多个BGP路由器之间的协调。
	3. BGP支持给予策略的选路。
	4. 可靠的传输。 （采用TCP协议）
	5. 路径信息。
	6. 增量更新。
	7. BGP支持无类型编制及VLSM方式。
	8. 路由聚合。
	9. BGP允许双方进行鉴别和认证。

[slide]

## 4.**BGP消息类型-- Open**

	
	1. **Open**： Open消息是BGP协议在TCP建立完成后的第一个消息，用于建立BGP对等体之间的连接关系。

[slide data-transition="slide1"]	
## 4.**BGP消息类型-- Keepalive**
	
	2.  **Keepalive**： 是一个类似探测、保活的消息。BGP会周期性的向对等体发送该消息，如果收到正确回复，那么就说明对等体还“或者”。

[slide data-transition="slide2"]	
## 4.**BGP消息类型-- Update**
	
	3.  **Update**： 用于对等体之间的路由信息交换。可以发布可达路由消息，也可以撤销不可达路由信息。
	
[slide data-transition="slide3"]	
## 4.**BGP消息类型-- Notification**

	4. **Notification**： 即当BGP检测到有错误状态时，向对等体发送该消息，通知本身有误，然后断开BGP连接。

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
