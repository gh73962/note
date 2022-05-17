# 基本资料

| 姓        名：   | 郭辉   | 手  机  号：   | 17612702804 |
| ---------------- | ------ | -------------- | ----------- |
| **出生日期：**   | 1993   | **是否在职：** | 是          |
| **经       验:** | 三年Go | **学    历:**  | 本科        |
# 职业技能

##### **主要岗位：**

* 面向用户服务端开发

##### **其他语言:**
* 有能力维护C++ PHP 项目
* 有能力维护python和shell脚本
##### **开源贡献:**

* github.com/xxjwxc/gormt
* github.com/golang/website
* github.com/golang/exp
##### **源码阅读:**

* go.uber.org/automaxprocs
* golang.org/x/sync/singleflight
* github.com/asim/go-micro/plugins/client/grpc/v3/
* runtime/map.go
* runtime/select.go


##### **论文学习:**

* 《In Search of an Understandable Consensus Algorithm》-- raft
* 《ImageNet Classification with Deep Convolutional Neural Networks》-- AlexNet
* 《Evaluating Large Language Models Trained on Code》-- codex

##### **其他:**

* 熟悉 TCP/IP 、HTTP等网络协议
* 熟练MySQL、Redis、ES等各种常用组件
* 熟悉常见的数据结构和算法

# 教育经历

| 2013.09 — 2017.06 | 武汉轻工大学 | 交通工程 | CET4  |

# 工作经历
| 时间              | 公司         | 职位         |
| ----------------- | ------------ | ------------ |
| 2021.04 — 至今    | 深圳巨世科技 | Golang工程师 |
| 2019.07 — 2021.04 | 深圳文思海辉 | Golang工程师 |
| 2018.09 — 2019.04 | 计算机考研   |              |
| 2017.07 — 2018.09 | 凌云科技集团 |              |


# 项目经验

## CrazyFox（深圳巨世科技）

### 项目描述：

海外游戏

### 项目职责：
- 微服务从0到1的设计与落地
- 用户服务、商城服务的迭代和维护
- 公共库的管理和review,制定规范
- 组件的调研和落地
- 负责第三方支付服务端接入（Google，Apple，Vivo）

#### 技术栈：
- 框架 go-micro
- 存储 MySQL（gorm），redis（v6），go-cache
- 监控 prometheus rpc协议 grpc  日志 elk 其他 k8s、etcd

#### 项目优化：

* 多处使用池化技术、减少资源的频繁分配与消耗：ants、sync.pool
* 增加XSS安全检查
* 项目中采用分库分表、读写分离（基于gorm）；SQL调优
* 调研本地内存缓存，减少大型静态配置的读取时间
* 设计错误响应渲染处理，提高开发效率
* 为业务引入prometheus监控业务健康
* 使用wire优化各类依赖

## 和平精英（微社区\公众号）服务端
### 项目职责：
- 微社区服务的迭代和优化
- 微社区\公众号 业务开发
- 各类亿级用户数据清洗和处理
- 维护 旧c++服务、PHP服务、py和shell、Hadoop脚本
- 调研组件和落地

#### 技术栈：
- 框架 Gin
- 存储 MySQL、redis
- rpc协议 trpc 其他 TGW Polaris 
  
#### 项目优化：
- 将python数据脚本模型重构，并用go编写，使其每日处理Hadoop下发数据时间从6小时缩短到半小时
- 优化基于go-kafka的脚本，使其每日增量数据处理时间缩短到2小时
- 设计中间层服务， 组合API依赖的数据，提高前端开发效率
- 基于go-playground/validator封装内部常用校验规则，提高开发效率