# python 基础配置
为了区分项目所需要的python库，避免安装多余的库，我们可以选择使用python的虚拟环境，对于python来说现在有三个流行形式去解决虚拟环境这个问题

1.  virtual environment
2. Piping
3. ancoanda  

## virtual environment
```python
# 安装虚拟环境
pip3 install env

# 创建虚拟环境
python3 -m venv file-path/file-name

# 在windows上激活环境
file-path/file-name\Scripts\active.bat
# 在macos上激活环境
source file-path/file-name/bin/active.sh
```

使用pip管理包
在pipenv和venv两个虚拟环境管理中
```bash
# 包搜索
pip search requests
# 包安装
pip install requests
# 指定包版本安装
pip install requests==2.6.0
# 升级包
pip install --upgrade requests
# 卸载包
pip uninstall reuquests
# 显示包信息
pip show requests
# 包列表
pip list


# 列出所有包到文件
pip freeze > requirements.txt
# 根据文件安装包
pip install -r requirements.txt
```

## 安装特定版本的pip
下载pip包,安装pip3则用python3执行，安装pip2则用python2执行
```bash
curl -o get-pip.py 'https://bootstrap.pypa.io/get-pip.py'
# 安装pip2
sudo python2.7 get-pip.py

# 安装py3
sudo python3 get-pip.py

# pip升级
python -m pip install --upgrade pip
```


### 编译安装python指定包
- 安装编译环境
```bash
yum -y groupinstall "Development tools"
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
yum install libffi-devel -y 
```

- 到官网下载python压缩包
```bash
wget https://www.python.org/ftp/python/3.7.0/Python-3.7.0.tar.xz 
```
- 压缩解压 编译安装
```bash
tar -xf Python-3.7.0.tar.xz
cd Python3.7.0
./comfigure --prefix=/usr/local/python3
make -j 8 && make install
```
- 创建软连接
```bash
ln -s /usr/local/python3/bin/python3.7 /usr/bin/python3
ln -s /usr/local/python3/bin/pip3 /usr/bin/pip 
```