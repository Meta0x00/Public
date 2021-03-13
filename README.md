# 记: 基于Ubuntu20.04搭建Prometheus+Grafana,实现容器级监控及可视化
## 背景概述: 
鉴于工作原因，前段时间搭建了一套websocket服务，但在实际部署中发现一些问题。由于网络稳定性原因及诸多不可控因素，经常发生ws连接偶发性中断，但服务本身又没有有效的健康检查机制。这样将对服务稳定性造成严重隐患。所以想着通过寻找引入第三方中间件，完成此任务。  
(起初，自己在本地写了个Shell脚本，循环监听进程，但毕竟太Low了，像贴膏药一样，根本拿不上台面，而且一旦Saas部署，可行性并不高)  
此时就想着要能有一套完整的监控解决方案，那该多呀~   
**T^T 咳咳，故事就这样开始了......**
## 选型分析：
Prometheus & Zabbix  
首先，观察下二者的架构图
![alt Prometheus](https://github.com/Meta0x00/Public/blob/master/img/Prometheus%E6%9E%B6%E6%9E%84%E5%9B%BE.png) 
![alt Zabbix](https://github.com/Meta0x00/Public/blob/master/img/Zabbix%E6%9E%B6%E6%9E%84%E5%9B%BE.png)  
### Prometheus：
- 基于Pull模式
- 采用TSDB
- 对应用层监控更加全面
- 支持云环境，自动发现容器，K8S提供对Prometheus的原生支持
- Alter-manager组件提供报警支持
- 时序库方便聚合分析，及UI展示  
- 集群化、持久化存储不方便、网络规划较复杂
### Zabbix：
- 基于Push模式
- 采用RDB
- 有完整的生态圈支持
- 无原生报警组件
- 对聚合数据分析及UI展示，支持欠佳  
### 对比综述：  
Prometheus在场景适配上，力压老牌的Zabbix。抛开功能完备性不谈(报警、聚合分析、数据渲染等)，瓶颈主要在DB。Zabbix默认使用常规的RDB，面对多写少读的真实监控场景，当QPS达到峰值(官方说是单机上限5000台)，RDB在IO处理上，必然影响整个系统的吞吐量，是性能的瓶颈。此时，Prometheus的时序库，完美适配了此种监控场景所需，虽然不及关系库检索时便捷，但重在时效性，碾压了各路SQL、NoSQL队友，使其在该领域脱颖而出。  
并不是说，Zabbix不够优秀，时间拨回到Zabbix出生的那个时代，它的设计绝对是非常前卫的，通吃当时的市场。但时间的巨轮一直向前滚动，面对新时代海量的数据，RDB自身的局限性，必然被新生代TSDB所取代。  
**T^T 科学家说: "要耍就耍时下最前沿的技术~"**  
# Demo
**Operations is not roadshow. Let's do it now.**

