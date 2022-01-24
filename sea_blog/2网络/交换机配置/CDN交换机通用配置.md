1.设置时间
clock timezone CN add 08:00:00
2.关闭版权信息
udno copyright-info enable



3.看不懂
```bash
ip vpn-instance 1002
 route-distinguisher 65513:1002
#
ip vpn-instance WD-SAD
 route-distinguisher 65513:30
#
ip vpn-instance office-106gw
 route-distinguisher 65513:8
#
ip vpn-instance office_kc
 route-distinguisher 65513:6
 description offic kc
#
ip vpn-instance subscribers
 route-distinguisher 65513:1
#
ip vpn-instance verycloud
 route-distinguisher 65513:3
#
ip vpn-instance wdcmn
 route-distinguisher 65513:2
 description WD CMN 
```

4.irf堆叠信息
```bash
 irf mac-address persistent timer
 irf auto-update enable
 undo irf link-delay
 irf member 1 priority 1
 irf mode normal
```