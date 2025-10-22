在文件 `/etc/docker/daemon.json` 中添加如下内容：
```json
{
    "registry-mirrors": [
        "https://docker.1ms.run",
        "https://docker.1panel.live/"
    ]
}
```