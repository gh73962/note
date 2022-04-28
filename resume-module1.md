# 基本资料

| 姓        名：   | 小于   | 手  机  号：   | 13112345678 |
| ---------------- | ------ | -------------- | ----------- |
| **出生日期：**   | 1992   | **是否在职：** | 是          |
| **经       验:** | Go/4年 | **学    历:**  | 本科        |
# 职业技能

##### **Web&&微服务：**

* 熟悉gin、echo、fiber各种web框架和go-kit、go-micro、go-zero等微服务框架
* 有完整的微服务和工程化开发经验：微服务框架(go-micro、go-zero)、服务注册与发现(etcd)、api网关(apisix)、分布式配置中心(apollo)、链路trace系统(jaeger、skywalking)、监控系统(promethus、grafana)、微服务治理(hystrix、sentinel-go)：熔断、限流

##### **Golang:**

* 具备扎实的语言基础
* 熟练掌握Golang数据结构(map、slice、channel等)底层原理
* 熟悉GPM运行时调度、内存分配、垃圾回收等机制
* 对Go并发(context、mutex、channel、goroutine等)、安全编程有深入的研究、理解和运用

##### **Source Code Read:**

* go.uber.org/automaxprocs
* golang.org/x/sync/singleflight
* github.com/asim/go-micro/plugins/client/grpc/v3/
* runtime/map.go
* runtime/select.go


##### **Paper Learning:**

* 《In Search of an Understandable Consensus Algorithm》--raft
* 《Implementing Remote Procedure Calls》
* 《ImageNet Classification with Deep Convolutional Neural Networks》-alexNet
* 《Evaluating Large Language Models Trained on Code》-- codex

##### **Other:**

* 熟悉 TCP/IP 、HTTP等网络协议
* 熟练MySQL、Redis、MQ(Kafka)、Elasticsearch等各种常用组件的运用并了解其原理
* 具备SQL调优能力并熟练运用调优工具
* 熟悉常见的数据结构和算法：查找、排序、树、图等相关算法
* 熟练运用Git版本控制工具并具有多分支和主干开发经验
* 了解熟悉各种分布式理论(BASE、CAP、2PC)，具有各种分布式技术的实战和分布式系统架构设计能力：负载均衡(hash、rand、roundrobin、load、session)、分布式锁、分布式事务(seata)、分布式任务调度系统、分布式缓存系统、分布式文件系统、分布式日志监控(ELK)
* 具备项目管理的能力：项目排期、严格项目质量把控、风控控制等

# 教育经历

| 2013.09 — 2017.06 | 北京大学 | 计算机与科学技术 |

# 工作经历

| 2017.06 — 至     今 | 北京移动联通小灵通公司 | Golang高级工程师 |
| ------------------- | ---------------------- | ---------------- |
| **工作内容：**      |                        |                  |

* 参与公司核心系统从0到1的设计与落地
* 负责核心服务的版本迭代和维护
* 日常负责指导同事

# 项目经验

## 项目1

### 项目描述：

> 最好用一两句话描述

### 项目职责：

> 可以写你项目中担任的角色和职责、使用的技术实现和收获了什么成果；
>
> 可以体现自己的优势能力和综合素质：比如为了提升效率，开发使用了自动化工具等；

#### 技术栈：

gorm、micro、redis、prometheus、elk、k8s、habor、helm

#### 项目优化：

* 多处使用池化技术、减少资源的频繁分配与消耗：ants(goroutine pool)、sync.pool(objectPool)、connPool
* 多处采用单例、适配器设计、工厂模式优化代码
* 项目中采用分库分表、读写分离；SQL调优
* 引入测试框架、代码自动生成等自动化工具提升业务开发效率

