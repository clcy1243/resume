# 项目列表
目录
=================
* [Go](#go)
    * [以 golang ast 为基础实现的「表达式解析执行服务」](#以-golang-ast-为基础实现的表达式解析执行服务)
    * [PHP 本地网关，类似 Side Car 概念](#php-本地网关类似-side-car-概念)
* [Java Spring](#java-spring)
    * [使用 Spring Webflux 实现的 「websocket 长链接转 MQ 网关」](#使用-spring-webflux-实现的-websocket-长链接转-mq-网关)
    * [主导设计搭建「WMS 仓储管理服务」](#主导设计搭建wms-仓储管理服务)
* [PHP](#php)
    * [使用 PHP 自研框架重构了「短租活动优惠计算服务」](#使用-php-自研框架重构了短租活动优惠计算服务)
    * [使用 Yii \+ Mongo 实现的「权限中心」（内部权限，用户数 3w\+ ）](#使用-yii--mongo-实现的权限中心内部权限用户数-3w-)


## Go
1. 以 golang ast 为基础实现的「表达式解析执行服务」
    - 
    - 前后端联合，提供复杂表达式配置，以在流程策略引擎中进行复杂判断处理
    - 使用 golang ast 解析、验证并优化表达式
    - 使用自定义 token 以映射 ast 解析出的语法 token
    - 使用 chan 作为通信渠道，以在复杂表达式深层次递归中进行并发执行，提高执行速度
    - 打通各个数据标签平台，以进行统一配置
2. PHP 本地网关，类似 Side Car 概念
    - 
    - 第一阶段：以 Rabbit MQ 为基础，使用 golang 搭建「消息服务平台」
        - 利用「死信队列」实现消费错误重试以及延时消息
        - 使用 swoole 开发 PHP 消费客户端，减少 PHP 消费时的线程开销
    - 第二阶段：开发 golang 守护进程作为 PHP 的本地网关
        - 利用消息平台作为感知渠道，实现服务状态感知
        - 服务于服务之间使用 golang 之间的长链接进行通信，和服务状态感知联动实现服务间负载均衡和无损上线
        - golang 和 PHP 之间使用 UNIX socket 通信，并且利用 swoole 提高PHP本身的负载能力
        - 去掉第一阶段的消费者客户端，凭借本地网关直接进行消费，减少 PHP 研发的负担
## Java Spring
1. 使用 Spring Webflux 实现的 「websocket 长链接转 MQ 网关」
    - 
    - 和客户端以 socket 构建通信
    - 和服务端以 kafka 构建通信
    - 基于业务需求以及运维现状，使用二次投递来保证消息触达
    - 通信图示例
    ```mermaid
    flowchart LR
    subgraph GW
        direction TB
        gateway1 <-.->|instance mq channel| gateway2
    end
    client <-->|socket channel| GW <-->|service mq channel| service
    ```
    - 详细设计[Websocket 网关设计](./socket-gateway.md) 

2. 主导设计搭建「WMS 仓储管理服务」
    - 
    - 按照业务领域，分为 存储、执行、单据 三大类，分别进行抽象构建
        - 存储：主管库存，引入金融理念，保证借贷双方账务准确性以及完备性
        - 执行：主管任务分发、执行
        - 单据：主管流程单据以及流转
    - 依据三大基础服务，搭建上层业务，依据业务特性，分为 入库、库内、出库 三个业务服务
    - 仓储功能模块化配置话，方便业务灵活组合，以应对业务的快速变化
## PHP
1. 使用 PHP 自研框架重构了「短租活动优惠计算服务」
    - 
    - 核心功能「用户日历范围内优惠活动计算」耗时从 999 分位 500ms+ 优化至 999 分位 10ms
    - 重新设计宽表实现的优惠配置，改为多策略配置，并且在 PHP 内使用 APCu 缓存相同的策略对象极大的减少重复的类加载和对象构建过程
    - 构建四级缓存机制，架设 DB - APCu - File - Redis - 客户端异步缓存，极大的提升了加载速度和用户体验
2. 使用 Yii + Mongo 实现的「权限中心」（内部权限，用户数 3w+ ）
    - 
    - 在设计上考虑读多写少的场景，以写入速度为代价，提高读取速度
    - 在数据权限上推出「结构树」配置以验证「节点树」结构是否合理
    - 利用 MQ 分发小粒度权限更新任务，以优化整体更新延迟
    - 数据结构设计如下
    
    ```mermaid
    flowchart TD
    subgraph "用户权限存储(运行时)"
        role2[角色] --> app3[应用]
        app3[应用] --> | 1:n | perm3[权限]
        app3[应用] --> | 1:n | node3[数据节点]
    end
    subgraph "用户权限结构(配置时)"
        user[用户] --> | 1:n | position[职位] --> | 1:n | role[角色]
        user[用户] --> | 1:n | role[角色]
        role[角色] --> app2[应用] --> | 1:n | pk2[权限包]
        app2[应用] --> | 1:n | gr2[权限组]
        app2[应用] --> | 1:n | perm2[权限]
        app2[应用] --> | 1:n | node2[数据节点]
    end
    subgraph 基本应用权限结构
        app1[应用] --> | 1:n | pk1[权限包]
        pk1[权限包] --> | 1:n | gr1[权限组]
        gr1[权限组] --> | 1:n | perm1[权限]
        pk1[权限包] --> | 1:n | node1[数据节点]
        node1[数据节点] --> | 1:n | node1[数据节点]
    end
    ```