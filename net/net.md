
# Net
## OSI (open system interconnection model)
| 层级  |    Name    |
| :---: | :--------: |
|   7   |   应用层   |
|   6   |   表示层   |
|   5   |   会话层   |
|   4   |   传输层   |
|   3   |   网络层   |
|   2   | 数据链路层 |
|   1   |   物理层   |

## TCP\IP
| 层级  |        Name        |           English           |     Example      |
| :---: | :----------------: | :-------------------------: | :--------------: |
|   4   |       应用层       |      Application layer      |   HTTP,DNS,FTP   |
|   3   |       传输层       |       Transport layer       | TCP,UDP,RTP,SCTP |
|   2   |     网络互连层     |       Internet layer        |      TCP\IP      |
|   1   | 网络访问（链接）层 | Network Access (link) layer |   以太网,WI-FI   |

## NAT
net address translation         

## TCP 
### 设计导向
稳定可靠遵循    
#### 可靠设计 (累计确认)
包皆有ID,收包确认按ID应答
### 基于窗口的流量控制
窗口偏移
### 拥塞控制
丢包不一定是堵塞了      
TCP BBR 拥塞算法
尽可能填满带宽,但不占用缓存
### 工作时序
### 快速重传

## Socket
