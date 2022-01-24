查看风扇状态
```bash
[H3C]display fan

 Slot 1

      FAN    1

      State    : FanDirectionFault

      Wind Direction    :Port-to-Power   ## 预期风向
 
      Prefer Wind Direction    :Power-to-Port  ## 当前风向
 
      FAN    2

      State    : FanDirectionFault

      Wind Direction    :Port-to-Power

      Prefer Wind Direction    :Power-to-Port

```


修改风向
fan prefer-direction slot 1 port-to-power