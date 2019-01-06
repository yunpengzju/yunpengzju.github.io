---
layout: post
title: Neutron代码赏析之VPN
description: Cisco VPN插件源码赏析
category: tech
---

### 源码简介
近来阅读了OpenStack Neutron的VPN代码，特别是Cisco的vpn插件，让我很受启发。故集中在此作一总结，希望能有更大的收获。源码选用Juno版Neutron代码(https://git.openstack.org/openstack/neutron)。本文大致分为两个部分：
1，Neutron中vpn业务的简介；  
2，Cisco vpn插件的亮点分析  

### Neutron中的VPN
Neutron Juno版中的VPN扩展主要包含四个资源对象：  
1, ipsec-policy  
2, ike-policy  
3, vpn-service  
4, ipsec-site-connection     
其中ipsec-policy和ike-policy均为协商加密算法相关的对象，配置了vpn通信加密机制的相关参数；vpn-service配置了本端子网，router等信息；而ipsec-site-connection对象则引用前三种对象，并包含对端子网信息，是实际建立的vpn连接。  

在创建或更新vpn-service及ipsec-site-connection时，Neutron-server侧的plugin会发rpc通知到vpn的agent具体下发vpn配置（下发到软件或硬件网关，建立vpn连接）。而对ipsec-policy和ike-policy的操作则只会对Neutron数据库做修改，不会通知Agent。  

相对而言Server侧的逻辑比较简单，下文主要聚焦Agent侧的代码分析。

### Cisco插件的读后感
##### 极为清晰的逻辑
（1）ipsec_driver接收到server端更新vpn服务的RPC消息后会进入同步数据流程（sync加了锁，不会出现线程冲突）  
<blockquote><code>@lockutils.synchronized('vpn-agent', 'neutron-')
def sync(self, context, routers)</code></blockquote>
（2）sync方法中分为四步，这里同一层次的方法抽象让代码可读性极高  
<blockquote><code>self.mark_existing_connections_as_dirty()
self.update_all_services_and_connections(context)
self.remove_unknown_connections(context)
self.report_status_internal(context)</code></blockquote>
（3）第一步中将所有vpnservice及connection标记为脏数据，在第二步成功更新过vpnservice之后会将标记位置回False；第三步删除所有非法的connection和vpnservice（在更新过后依然被标为dirty的数据）；第四步获取vpnservice及apices-site-connection的状态并上报给Neutron-server，修改DB中的状态数据（在cisco的Driver中，创建vpn连接以及获取状态均通过csr_client发送REST请求至Cisco Cloud Services Router完成）    
（4）CsrRestClient对象封装了Cisco Cloud Services Router的所有接口。隐藏了鉴权及发送请求的细节。只对ipsec_driver暴露了在router上vpn连接相关资源对象的CURD接口。从而使Neutron中的VPN组件与具体对接的外部网关设备解耦。

##### 注释与日志
对于服务端程序开发，日志是定位问题的主要手段。而Cisco vpn driver的日志记录非常精致，值得学习。  
1，每一步操作后，记录操作的对象的数量（注意日志的格式）  
```
LOG.debug(_("Mark: %(service)d VPN services and %(conn)d IPSec "
"connections marked dirty"), {'service': service_count,
'conn': connection_count})
```

2，关键事件要记录下操作的资源对象的唯一标示符   
`LOG.debug(_("Update: New VPN service %s detected"), vpn_service_id)`

3，什么是细节？细节就是日志中也会注意英语里单复数的语法  
```
LOG.debug(_("Sweep: Removed %(service)d dirty VPN service%(splural)s "
"and %(conn)d dirty IPSec connection%(cplural)s"),
{'service': service_count, 'conn': connection_count,
'splural': 's'[service_count == 1:],
'cplural': 's'[connection_count == 1:]})
```
4，捕捉到异常要记录error日志

```
except (CsrUnknownMappingError, CsrDriverMismatchError) as e:
    LOG.exception(e)
```
5，对外部系统的操作，又info级别的日志记录操作结果
`LOG.info(_("SUCCESS: Created IPSec site-to-site connection %s"),
conn_id)`
关于注释，这里就不多说了。每个方法都有doc string注释，关键的流程有详细的解释性描述。在方法圈复杂度不高的情况下，这样的注释已经足以辅助理解了。

##### 测试驱动开发
Cisco vpn-driver编写单元测试的难点在于与网关设备的交互（大量REST请求），如何模拟这些请求是测试的关键。可以看到，单元测试的代码行数与功能代码的行数是相近的。用例详见neutron/tests/unit/services/vpn/device_drivers目录