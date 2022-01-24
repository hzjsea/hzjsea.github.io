# ansible-playbook
## 根据文件来实现

1. 控制包版本
```bash
包版本存储在role/cfg.yml下面
     - app_version:  # 控制包版本         ->(明文
     - app_md5 加密   # 控制包版本下载地址  gitlab app_md5
     - conf_hash     # 控制配置文件       也是gitlab上面下载的？  

# 如果没有conf配置文件，比如配置文件随包下载的，那就不需要conf配置文件了
# 只有在配置conf管理中心的文件 才需要下conf_hash
```
2. 包存储地址初始化   init文件
```bash
# /roles/common/tasks/init.yml
---
- name: check and create dir
  file: path={{ item }} state=directory mode=0755
  with_items:
    - "{{ backup_path }}" # backup_path: "/disk/ssd1/backup"
    - "{{ main_path }}"  # main_path: "{{ prefix_path }}/{{ project }}"
    - "{{ work_path }}"  # work_path: "/tmp/ansible"
    - "{{ log_path }}"  # log_path: "/disk/ssd1/logs"

# prefix_path: "/usr/local"
# project sa-agent
```

3. 获取本地包到目标主机 app文件 
```bash
# /roles/common/tasks/app.yml
---
   - name: fetch app package
     fetch_pack: "project={{ project }} tar={{ app_tgz }} dest={{ work_path }} cdn={{ fetch_from_cdn | default(true) }}"
   - name: unarchive app package
     unarchive:
       src: "{{ work_path }}/{{ app_tgz }}"
       dest: "{{ prefix_path }}"
       remote_src: yes

```

4. 更新包版本信息 version文件
```bash
# /roles/common/tasks/version.yml
---
- name: write current version to .version
  backup: role={{ project }} version={{ app_version }}-{{ conf_hash }} update=true

```

5. 回滚版本 rollback文件
```bash
# /roles/common/tasks/rollback.yml
---
- name: get last version from .version
  backup: role={{ project }} version={{ app_version }}-{{ conf_hash }}
- name: check backup file
  stat: path={{ backup_path }}/{{ project }}-{{ backup_version }}.tar.gz
  register: backup_file
- fail:
    msg: backup_file {{ backup_version }} is not exists
  when: not backup_file.stat.exists
- name: rollback
  unarchive: src={{ backup_path }}/{{ project }}-{{ backup_version }}.tar.gz dest={{ prefix_path }}/ copy=no
- name: replace backup_version to current_version
  backup: role={{ project }} version={{ app_version }}-{{ conf_hash }} replace=true

```


## 根据流程标签来实现

### install流程
install的时候
1. 初始化init
2. 安装并解压app
3. 本地版本控制version

### rollback流程
仅回滚

### app流程
1. 安装并解压app
2. 备份backup
3. 本地版本控制机version



### 项目流程
几个固定的操作
```bash
1. 回滚rollback
2. 安装并解压app
3. 初始化init
4. 备份backup
5. 本地版本控制version
```

项目流程1
```bash
root权限下，在/usr/local目录下新建sa目录,mkdir /usr/local/sa 
# 初始化目标主机环境 init.yml  -t  install 

将sa-agent-linux64.tar.gz及SA.cert文件上传到 /usr/local/sa目录下。
解压tar.gz文件，tar -zxvf sa-agent-linux64.tar.gz

# 安装并解压 app   -t install

如主机上部署了数据库，需修改/usr/local/sa/sa-agent/conf/config-db.properties文件，若没部署数据库，此步骤跳过。

# start  -t start
运行./server.sh start指令，
完成后运行./checkAsset.sh  
完成后运行。/checkSafe.sh

# 回滚 rollback 
# backup
# version
```


