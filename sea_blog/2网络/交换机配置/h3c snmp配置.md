h3c文档地址http://www.h3c.com/cn/d_201412/847683_30005_0.htm

## snmp配置流程
snmp分为v1/v2版本的配置和v3的配置方法，这里先只记录v3的配置




## 具体配置
```bash
# 开启snmp-agent服务
snmp-agent
# 设置本地SNMP实体的引擎ID
snmp-agent local-engineid 800063A203
# 设置访问权限
snmp-agent community write hgE6ofdZ3b
# 启用v2 v3版本
snmp-agent sys-info version v2c v3
```