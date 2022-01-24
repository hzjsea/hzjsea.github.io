## 清空配置
```bash
<h3c> reset saved-configuration
# 或者使用
<h3c> restore save-configuration
# 配置会被清除，是否确认
The saved configuration file will be erased. Are you sure? [Y/N]: Y
# 重启
<h3c> reboot 
# 开始检查启动配置文件，请等待
# 当前配置再重启后会被删除，是否保存当前配置 N
Start to check configuration with next startup configuration file, please wait.........DONE!
Current configuration may be lost after the reboot, save current configuration? [Y/N]:N
# 设备即将被重启，是否确认
This command will reboot the device. Continue? [Y/N]: Y



```