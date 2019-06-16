### 传输层服务（协议）

* 提供了应用进程之间的一种逻辑通信机制（位于网络层之上，依赖网络层，对网络层服务进行增强）（端到端）（网络层提供的是主机之间的逻辑通信机制）
* 发送方：将应用递交的消息分成一个或多个的Segment，并向下传给网络层
* 接收方：将接收到的Segment组装成消息，并向上交给应用层
* 提供多种协议：TCP/UDP
TCP：可靠，按序，提供拥塞控制，流量控制，面向连接

UDP：不可靠（尽力而为）

均不提供延迟和带宽的保障
 

#### 多路复用/分用

如果某层的一个洗衣对应直接上层的多个协议/实体，则需要多路复用/分用

1. 每个数据报携带IP地址，目的IP地址
2. 每个数据报携带一个传输层的段（segment）
3. 每个段携带源端口号和目的端口号 
4. 主机收到segment后传输协议提取IP地址和端口号，将segment导向相应socket
```
源端口号|目的端口号
其他头部信息
应用数据

TCP段格式
```

* 多路复用（发送端，从多个Socket接收数据，为每块数据封装上头信息，形成segment交给网络层）
* 多路分用（接收端，根据头部信息将接收到的segment交给正确的Socket即不同进程）

UDP分用：二元组标识（目的IP和目的端口号）
TCP分用：四元组标识（源IP,源端口，目的IP，目的端口）通过源端口号区分不同连接
TCP分用(多线程)：通过进程创建多个线程进行服务

#### UDP(User Datagram Protocol)(Best effort尽力而为)（和网络层一样不可靠）
* 复用/分用
* 简单的错误校验
* 无连接（丢失）（非按需到达）（延迟显著减少）
* 在UDP上实现可靠数据传输：
在应用层增加可靠性机制，应用特定的错误恢复机制
```
源端口号|目的端口号
length|chekcsum
其他头部信息
应用数据

UDP段格式
```
* UDP校验和（checksum）:检测UDP段在传输中是否发生错误

发送方：将段的内容视为16-bit整数，计算所有整数的和，高位要被加到低位，将得到的值按位求反，得到校验和放入校验和字段
```
checksum 计算

 1110011001100110
+1101010101010101
11011101110111011(发生进位)
 1011101110111100（低位+1）
 0100010001000011（按位取反得出checksum）
```
接收方：计算收到段和，将其和校验和比对

#### 可靠数据传输机制（不错，不丢，不乱）（Reliable Data Transfer可靠数据传输RDT）
* 可靠数据传输协议Rdt1.0

假设底层信道是完成可靠的：

不会发生错误(bit error)

不会丢弃分组

```
rdt_send()//上层网络应用单项的效用下层可靠协议
udt_send()
rdt_rcv()
deliver_data()//下层协议将结果单向交付给上层网络
```

* 可靠数据传输协议Rdt2.0

可能产生位错误

如何确认可能出现位翻转：

利用校验和检测位错误

 如何从错误中恢复：
 
 确认机制（Acknowledgements,ACK）接收方显式地告知发送方分组已正确接收
 
 NAK:接收方显式告知发送方数组有错误，发送方收到NAK后重传分组

基于这种重传机制的rdt协议称为ARQ(Automatic Repeat reQuest)协议

Rdt2.0中引入的新机制：
1. 差错检测
2. 接受反馈:ACK/NAK
3. 重传

* 可靠数据传输协议Rdt2.1
2.0存在ACK/NAK消息发生错误/被破坏问题：

为ACK/NAK增加校验和，检错并纠错（不合适）

发送方搜到被破坏的ACK/NAK时不知道接收方发生了什么，添加额外的控制消息（不合适）

为ACK/NAK增加校验和，如果ACK/NAK坏掉，发送方重传，但不能简单重传，会产生重复分组（合适），通过增加序列号解决重复分组问题（发送方给每个分组增加序列号，接收方丢弃重复分组）

* 可靠数据传输协议Rdt2.2（取消NAK）

接收方：通过ACK告知最后一个被正确接收的分组，在ACK消息中显示的加入被确认分组的序列号

发送方：收到重复的ACK后，采用与收到NAK消息相同的动作：重传当前分组

* 可靠数据传输协议Rdt3.0
 假设信道会发生错误，也可能丢失分组
 
 解决丢失问题：
 发送方等待“合理”时间（超时），如果没有收到ACK就重传（需要定时器）
 
 如果分组或ACK只是延迟而不是丢失，通过系列号机制解决重传产生的重复（接收方需要在ACK中显式告知所确认的分组）
 
 
 由于停等操作，性能差，利用率0.00027
 
 通过流水线机制打破效率低下：
 1. 允许发送方在收到ACK之前连续发送多个分组（更大的序列范围）（发送方和接收方需要更大的存储空间以缓存分组）
 2. 要实现流水线机制需要“滑动窗口协议（Sliding-window protocol）（GBN,SR）”:
 
 窗口：
 
 允许使用的序列范围
 
 窗口的尺寸为N:最多有N个等待确认的消息
 
 
滑动窗口：

随着协议的运行，窗口在序列号空间内向前滑动
```
||||||||||||||
   ↑------↑
箭头左边的是已完成ack的，箭头中间包含发送未响应和可使用的，箭头右边是不可使用的   
```

GBN协议(Go-Back-N)

发送放：
1. 返祖头包含k-bit序列号
2. 窗口尺寸为N，最多允许N个分组未确认
3. 采用累积确认机制，收到ACK包含N，意思是确认到N的分组包含N都已经被正确接收（可能重复）
4. 为空的分组设置定时器
5. 超时事件：GBN选择重传序列号大于等于N且还未收到ACK所有分组
6. 存在乱序到达的分组：GBN选择直接丢弃

```
例题：

问：采用GBN协议，发送方返送编号为0-7的帧，当计时器超时，发送方收到0，2，3号帧的确认，则需要重发帧数是多少，分别是哪几个?

答：GBN使用累积确认，所以此时需要重发4个，依次为4，5，6，7（3之前包括3都确认了，所以从4开始重发）

```

SR协议(Selective Repeat)

GBN存在重传很多分组缺陷

SR存在无法识别重复的序列号缺陷（解决方法把序列号扩大，序列号大于等于发送方窗口长度+接收方窗口长度）


接收方：
1. 对每个分组单独进行确认
2. 设置缓存机制，缓存乱序到达的分组
3. 接收方存在窗口（期望收到还没收到，乱序到达分组，可以接收的）
[![V7F2uR.md.png](https://s2.ax1x.com/2019/06/16/V7F2uR.md.png)](https://imgchr.com/i/V7F2uR)

[![V7FLDI.md.png](https://s2.ax1x.com/2019/06/16/V7FLDI.md.png)](https://imgchr.com/i/V7FLDI)
发送方：
1. 只需要重传那些没有收到ACK的分组（为每个分组设置定时器）

#### 流量控制机制
#### 拥塞控制机制