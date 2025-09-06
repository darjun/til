# 永久更换 pip 源

这是最根本的解决方案。将默认的源更换为国内的镜像源，速度会有质的飞跃。常见的国内镜像源有：

清华 tuna： `https://pypi.tuna.tsinghua.edu.cn/simple`

阿里云： `http://mirrors.aliyun.com/pypi/simple/`

中国科技大学： `https://pypi.mirrors.ustc.edu.cn/simple/`

豆瓣 douban： `http://pypi.douban.com/simple/` (有时稳定性稍差)

华为云： `https://repo.huaweicloud.com/repository/pypi/simple/`


在用户目录下（C:\Users\你的用户名\）创建一个 pip 文件夹，然后在其中创建 pip.ini 文件。

步骤如下：

1. 打开命令提示符 (CMD) 或 PowerShell，依次执行以下命令：

```bash
# 进入用户目录
cd %USERPROFILE%

# 创建 pip 目录
mkdir pip

# 进入 pip 目录
cd pip
```

2. 创建并编辑 pip.ini 文件：

* 你可以用记事本等任何文本编辑器创建这个文件。

* 直接运行 notepad pip.ini，然后在弹出的记事本中输入以下内容（以清华源为例）：

```ini
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = pypi.tuna.tsinghua.edu.cn
```
* `index-url` 是源的地址。

* trusted-host 表示信任该主机，不然使用 HTTPS 时可能会报信任错误。

3. 保存文件。之后，你所有使用 pip 的命令都会自动从这个镜像源下载，速度飞快。