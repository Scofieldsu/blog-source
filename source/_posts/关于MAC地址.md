---
title: 设备MAC地址随机化与WiFi探针
date: 2017-12-15 20:47:23
categories: 编程
tags: [伪MAC,探针]
---

# 关于MAC 地址（Media Access Control Address）

---                                                  

## 1. MAC地址共48位（6个字节），以十六进制表示。前24位由IEEE决定如何分配，后24位由实际生产该网络设备的厂商自行指定。


![mac](https://github.com/Scofieldsu/blog-source/tree/main/source/images/MACadress.png)

图片来源  wikipedia链接：https://en.wikipedia.org/wiki/MAC_address

---

## 区分方法

#### 通过B1位区分一些修改过的MAC地址

这个6个字节中的最高有效字节中的最低有效位（b0）用来标识 unicast,和mulcast，即单播 和多播  而**次最低有效位（b1）则用来标识 universally administered address 和  locally administered address**  其中：universally administered address 是指烧录在固件中由厂商指定的地址，也也即大家通常所理解的MAC地址，  而 locally administered address 则是指由网络管理员为了加强自己对网络管理而指定的地址，由定义可知， locally administered address的U/L位要设置成1.所以 要表示 locally administered address的话，那 MAC 地址的第一个字节应该是 0x02, 因此不能够把MAC地址改成其它已经被厂商占用的 universally administered address了。

但是通常情况下，很多人都不会遵守上面的约定。因为 MAC地址通常只能在局域网里发挥作用，因为在网络上传输，MAC地址是会被不断的替换掉的。所以即使你用了 已经被厂商占用的 universally administered address也不用担心会产生冲突。  说到着就更清楚了吧。说到底， **locally administered address 和 universally administered address的区别在于是直接由生产的时候确定的还是由你自己修改过的。**

原文链接：http://blog.sina.com.cn/s/blog_9950926401019kzx.html

#### 通过前3个字节查询对应注册厂商

IEEE注册厂商查询，通过匹配MAC的前3个字节查找对应厂商，如无对应则可认为是一些低级的伪MAC地址。但这个注册厂商列表是公开的，一般的模拟MAC地址时，前3个字节从列表中取即可，而且这样伪造的MAC地址也不存在上面通过B2位判断的问题。

IEEE官网查询地址（可以下载oui.csv）：https://regauth.standards.ieee.org/standards-ra-web/pub/view.html#registries

在线oui对应列表：http://standards-oui.ieee.org/oui/oui.txt

#### 通过流量判断
CSNA网络分析论坛有一种说法是 通过流量判断：伪造的IP或MAC地址，一般都只有单向流量，而且你可以看到其接收或发送的数据包都非常的少。

原文链接：http://www.csna.cn/archiver/tid-2040.html


#### 通过MAC地址前3字节+后3字节

这种整体判断MAC真伪方式暂时还没找到。

----

##### 其他相关链接

- 知乎问题《iOS 8 设备随机 MAC 地址躲避 Wi-Fi 热点的记录追踪，技术上是怎么实现，有何影响？》https://www.zhihu.com/question/24094236


# ios8 MAC随机化分析

在iPhone 5s中发现了MAC随机化（细节如下），但在iPhone 5和iPad Mini中却没有。我怀疑这与新一代iPhone的操作系统架构差异有关。


#### *在iPhone 5s中，MAC随机化仅在以下情况下发生：*

- 手机处于睡眠模式（显示关闭，未被使用）
- Wi-Fi应该打开但不关联
- 位置服务应在隐私设置中关闭

#### *在上述条件下，iPhone5s将在探测请求中使用随机MAC，具有以下特征：*

- 随机化的MAC是本地管理的MAC。

- **在电话显示屏关闭后大约120-150秒钟，手机将传送第一批具有随机MAC的探测请求**。

- 在2.4 GHz和5 GHz频段内，批次内和所有通道内的所有探测请求都使用相同的随机MAC地址。

- **下一批次的探测请求会随着批次之间的升级间隔而增加，最大间隔约为385秒，然后下一批次再次以大约120-150秒作为初始时间。跟踪时间为1小时**。

- 所有这些探测请求都使用相同的随机MAC地址。

- **探测请求中使用的随机MAC地址在每次手机被激活并随后进入睡眠模式时都会更改。意味着每个新的睡眠周期都使用一个新的随机MAC**。

- 处于睡眠模式的探测请求不要求特定的SSID（称为空探测）。这似乎是一个额外的隐私功能，以防止在手机的无线配置文件中的SSID显示在睡眠模式。

- 无论手机是否在充电，MAC随机化都会发生。

## 结论

**测请求中使用的随机MAC地址在每次手机被激活并随后进入睡眠模式时都会更改。意味着每个新的睡眠周期都使用一个新的随机MAC**。

原文地址：https://blog.mojonetworks.com/ios8-mac-randomization-analyzed/

---

相关：https://blog.mojonetworks.com/ios8-mac-randomgate/

专利《一种获取WiFi终端真实MAC地址的装置及方法》：   https://www.google.com/patents/CN107094293A?cl=zh

---


# 关于探针

### WiFi使用的网络协议，WiFi采用的是IEEE802.11协议集，此协议集包含许多子协议。其中按照时间顺序发展，主要有：

（1）802.11a，

（2）802.11b，        

（3）802.11g

（4）802.11n，。

在网络通信中，数据被封装成了帧，帧就是指通信中的一个数据块。但是帧在数据链路层传输的时候是有固定格式的，不是随便的封装和打包就可以传输，大小有限制，最小46字节，最大1500字节所以我们必须按照这个规则来封装。

下面802.11的帧结构：

| 前导码 | 前定界符 | 目的地址 | 源目的地址 | 长度字段 | 数据字段 | 校验字段 |
| - - - -|- - - - -| - - - - | - - - - - | - - - - | - - - - -| - - - - |
| 7B     |  1B     |   6B    |    6B     |    2B   |  46-1500 |   4B    |

在802.11中的帧有三种类型：管理帧（Management Frame，例如Beacon帧、Probe Request帧）、控制帧（Control Frame，例如RTS帧、CTS帧、ACK帧）、数据帧（Data Frame，承载数据的载体，其中的DS字段用来标识方向很重要）。帧头部中的类型字段中会标识出该帧属于哪个字段。

我们主要介绍下WiFi探针技术相关的几种帧：

**1.管理帧：**

- *BeaconFrame*：信标帧，是相当重要的维护机制，主要来宣告某个AP网络的存在。定期发送的信标，可让移动WiFi设备得知该网络的存在，从而调整加入该网络所必要的参数。在基础网络里，AP必须负责发送Beacon帧，Beacon帧所及范围即为基本服务区域。  在基础型网络里，所有沟通都必须通过接入点，因此WiFi设备不能距离太远，否则无法接收到信标。

   下图是帧格式：

   ![BeaconFrame](https://github.com/Scofieldsu/blog-source/tree/main/source/images/beaconframe.png)


- *Probe Request*：探测请求帧，WiFi设备将会利用Probe Request帧，扫描所在区域内目前有哪些802.11网络。

    下图是帧格式：
    ![probe](https://github.com/Scofieldsu/blog-source/tree/main/source/images/probe.png)


**2.数据帧**：

- *Data*：数据帧，当接入点要送出一个帧给WiFi设备但是不必确认之前所传送的信息时，就会使用标准的数据帧。标准的数据帧并不会征询对方是否有数据待传，因此不允许接收端传送任何数据。无竞争周期所使用的纯数据（Data-Only)帧和无竞争周期所使用的数据帧完全相同。

![wifi-run](https://github.com/Scofieldsu/blog-source/tree/main/source/images/wifi-run.png)


就像图中描述的一样，我们的WiFi探针其实就是一个AP，它定时的向自己的四周广播发送Beacon帧，用来通知附近的WiFi设备，AP是存在的，（好比它一直在向周围喊着，我在这里，大家快来连接我啊）。


我们的WiFi设备，手机，平板电脑等，也不停的发送着probe帧，去寻找附近可用的AP。在probe帧的介绍中就我们可以看到probe帧包含了设备的mac地址，当我们的AP接收到probe帧之后就获取了这个设备的MAC地址，而这个AP就是我们的WIFI探针。因此只要在WiFi探针覆盖区域内的设备打开着WiFi，探针就能收集到他的MAC地址。

----
关于探针安全

- 探针所收集的数据内容

    > 我们来看看WiFi探针设备究竟会收集什么信息？前面我们已经看到了，在不连接WiFi的情况下，移动设备只会发送probe帧，此时我们并不能通过探针访问网络进行数据传输，探针仅仅只能接收到WiFi设备发送的probe帧，收集probe帧携带的MAC地址，所以此时我们收集到信息是绝对无关用户个人信息和设备上其他信息的。

- 探针不提供上网功能，WiFi提供上网功能

    >探针不提供上网功能，所以用户与探针之间的数据流量仅限在探针发送广播，移动设备发送一次连接请求，而手机号码，用户姓名，性别等等个人信息和设备的详细信息都不包含在里面。但是WiFi就不一样了。因为用户可以用它来上网，可以说用户所有流量都是WiFi可以看到的。而流量的内容里面有很多是关于用户个人信息的。比如用户在网上填写市场调查问卷，可能就有一些用户信息，比如性别，年龄，手机号码等。甚至那种别有用心的人提供的WiFi会估计记录用户的一些敏感信息，比如用户上网时输入的密码等等。这也是安全相关人士经常提到的不要随便上别人的WiFi的原因。

- 探针的数据处理

    >由于探针本身设计仅仅是探测周边有些什么设备，因此并不产生大量数据，设计的时候就不会将收集到的数据存储在本身，而是通过有线连接直接发送到中心服务器上，这样即使有恶意的人将探针取走，也不能获得探针收集到的信息。同时有线连接也保证数据传输过程不容易通过电磁波的形式被监听和窃取。中心服务器一般都是在IDC机房里，而要进入IDC机房是需要经过IDC层层许可的。因而不论是数据的传输还是存储，探针的数据都是安全的。


原文链接：http://blog.51cto.com/11926581/1834693
