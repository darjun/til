## 基本概念
tcpdump 工作原理：

  网络数据包
      │
      ▼
  ┌─────────────┐
  │  网络接口    │  (eth0, wlan0...)
  └──────┬──────┘
         │
         ▼
  ┌─────────────┐
  │  BPF 过滤器  │  (Berkeley Packet Filter)
  └──────┬──────┘
         │ 匹配的包
         ▼
  ┌─────────────┐
  │   tcpdump   │  显示/保存
  └─────────────┘

## 基本语法
```bash
tcpdump [选项] [过滤表达式]

# 基本结构
tcpdump -i eth0 -n -v 'host 192.168.1.1 and port 80'
         │       │   │   └─────── 过滤表达式
         │       │   └─────────── 详细模式
         │       └─────────────── 不解析主机名
         └─────────────────────── 指定网卡
```

## 常用选项
```bash
# 网卡相关
-i eth0          # 指定网卡
-i any           # 监听所有网卡

# 显示相关
-n               # 不解析 IP 为主机名
-nn              # 不解析 IP 和端口名
-v               # 详细输出
-vv              # 更详细
-vvv             # 最详细
-X               # 显示十六进制和 ASCII
-XX              # 显示包括以太网头的十六进制
-A               # 只显示 ASCII（适合看 HTTP）
-q               # 简洁输出
-e               # 显示以太网头（MAC地址）
-t               # 不显示时间戳
-tttt            # 显示完整时间戳

# 数量相关
-c 100           # 只抓 100 个包
-s 0             # 抓取完整包（默认可能截断）
-s 1500          # 指定抓取长度

# 文件相关
-w file.pcap     # 保存到文件
-r file.pcap     # 读取文件
-C 100           # 文件超过 100MB 自动切割
-W 5             # 配合 -C，最多保留 5 个文件

# 其他
-D               # 列出所有可用网卡
-l               # 行缓冲（实时输出）
-p               # 不使用混杂模式
```

## 过滤表达式

### 基础过滤
```bash
# 按主机
tcpdump host 192.168.1.1          # 源或目标
tcpdump src host 192.168.1.1      # 只看源
tcpdump dst host 192.168.1.1      # 只看目标

# 按网段
tcpdump net 192.168.1.0/24
tcpdump src net 10.0.0.0/8

# 按端口
tcpdump port 80
tcpdump src port 8080
tcpdump dst port 443
tcpdump portrange 8000-9000       # 端口范围

# 按协议
tcpdump tcp
tcpdump udp
tcpdump icmp
tcpdump arp
tcpdump ip6                       # IPv6
```

### 逻辑组合
```bash
# AND
tcpdump host 192.168.1.1 and port 80
tcpdump host 192.168.1.1 && port 80   # 同上

# OR
tcpdump port 80 or port 443
tcpdump port 80 || port 443

# NOT
tcpdump not port 22
tcpdump ! port 22

# 组合使用
tcpdump 'host 192.168.1.1 and (port 80 or port 443)'
tcpdump 'src 192.168.1.1 and not dst port 22'
```

### 高级过滤
```bash
# TCP 标志位过滤
tcpdump 'tcp[tcpflags] & tcp-syn != 0'    # SYN 包
tcpdump 'tcp[tcpflags] & tcp-fin != 0'    # FIN 包
tcpdump 'tcp[tcpflags] & tcp-rst != 0'    # RST 包
tcpdump 'tcp[tcpflags] == tcp-syn'        # 只有 SYN（无 ACK）
tcpdump 'tcp[tcpflags] == (tcp-syn|tcp-ack)'  # SYN+ACK

# 包大小过滤
tcpdump 'len > 1000'              # 大于 1000 字节
tcpdump 'len < 100'               # 小于 100 字节

# IP TTL 过滤
tcpdump 'ip[8] < 5'              # TTL 小于 5

# 过滤分片包
tcpdump 'ip[6:2] & 0x1fff != 0'

# HTTP GET 请求
tcpdump 'tcp port 80 and (tcp[((tcp[12:1] & 0xf0) >> 2):4] = 0x47455420)'

# ICMP 类型
tcpdump 'icmp[icmptype] == icmp-echo'      # ping 请求
tcpdump 'icmp[icmptype] == icmp-echoreply' # ping 响应
```

## 常用场景分析

### 网络诊断
```bash
# 抓取所有流量（调试用）
tcpdump -i any -nn

# 查看与某主机的通信
tcpdump -i eth0 -nn host 8.8.8.8

# 监控 DNS 查询
tcpdump -i any -nn port 53

# 查看 DNS 查询详情
tcpdump -i any -nn -v port 53

# 检测网络扫描（大量 SYN 包）
tcpdump 'tcp[tcpflags] == tcp-syn'

# 查看 ICMP（ping）
tcpdump icmp

# 抓取某网段所有流量
tcpdump net 192.168.1.0/24
```

### HTTP分析
```bash
# 抓取 HTTP 流量并显示内容
tcpdump -i any -A -s 0 port 80

# 只看 HTTP 请求头
tcpdump -i any -A -s 0 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'

# 抓取 HTTP 和 HTTPS
tcpdump -i any port 80 or port 443

# 实时查看 HTTP 请求（grep 过滤）
tcpdump -i any -A -s 0 port 80 | grep -E "GET|POST|Host:"
```

### 保存和分析
```bash
# 保存到文件（给 Wireshark 分析）
tcpdump -i eth0 -w capture.pcap

# 后台抓包
tcpdump -i eth0 -w capture.pcap &

# 按大小滚动保存
tcpdump -i eth0 -C 100 -W 10 -w capture.pcap
# 生成 capture0.pcap, capture1.pcap ... 最多10个，每个100MB

# 按时间抓包（抓60秒）
timeout 60 tcpdump -i eth0 -w capture.pcap

# 读取并过滤已保存的文件
tcpdump -r capture.pcap host 192.168.1.1
tcpdump -r capture.pcap -nn port 80
```

### 安全排查
```bash
# 检测 ARP 欺骗
tcpdump -i eth0 arp

# 查看异常连接（大量 RST）
tcpdump 'tcp[tcpflags] & tcp-rst != 0'

# 检测端口扫描
tcpdump 'tcp[tcpflags] == tcp-syn' -nn

# 抓取密码（明文协议）
tcpdump -i eth0 -A port 21    # FTP
tcpdump -i eth0 -A port 23    # Telnet
tcpdump -i eth0 -A port 110   # POP3

# 查看 SSH 流量（加密，只能看握手）
tcpdump -i eth0 port 22 -nn
```

### 性能分析
```bash
# 统计各协议流量
tcpdump -i eth0 -q -nn

# 查看大包（可能影响性能）
tcpdump 'len > 1400'

# 抓取特定时间段
tcpdump -G 3600 -W 24 -w capture_%Y%m%d_%H%M%S.pcap
# 每小时一个文件，保留24个
```

## 输出解读
```bash
# 典型输出
14:32:01.123456 IP 192.168.1.100.52341 > 93.184.216.34.80: 
    Flags [S], seq 1234567890, win 65535, length 0

# 字段说明
14:32:01.123456          → 时间戳
IP                       → 协议类型
192.168.1.100.52341      → 源IP.源端口
93.184.216.34.80         → 目标IP.目标端口
Flags [S]                → TCP 标志
seq 1234567890           → 序列号
win 65535                → 窗口大小
length 0                 → 数据长度
```

### TCP Flags含义
```bash
[S]    = SYN           建立连接
[.]    = ACK           确认
[S.]   = SYN+ACK       连接响应
[P.]   = PSH+ACK       推送数据
[F.]   = FIN+ACK       关闭连接
[R.]   = RST+ACK       重置连接
[R]    = RST           强制重置
[FP.]  = FIN+PSH+ACK
```

### 三次握手示例
```bash
tcpdump -i eth0 host example.com and port 80

# 输出
10:00:01.001  client.54321 > server.80:  Flags [S]          # ① SYN
10:00:01.002  server.80 > client.54321:  Flags [S.]         # ② SYN+ACK
10:00:01.003  client.54321 > server.80:  Flags [.]          # ③ ACK
10:00:01.004  client.54321 > server.80:  Flags [P.] len 80  # 发送数据
10:00:01.005  server.80 > client.54321:  Flags [.]          # 确认
10:00:01.100  server.80 > client.54321:  Flags [P.] len 512 # 响应数据
10:00:01.101  client.54321 > server.80:  Flags [F.]         # 关闭
```

## 使用技巧
### 配合其他工具
```bash
# 实时统计流量
tcpdump -i eth0 -nn -q | awk '{print $3}' | sort | uniq -c | sort -rn

# 提取 HTTP Host
tcpdump -i any -A port 80 | grep "Host:"

# 保存并实时查看
tcpdump -i eth0 -w - | tee capture.pcap | tcpdump -r -

# 远程抓包传给本地 Wireshark
ssh user@remote "tcpdump -i eth0 -w -" | wireshark -k -i -

# 过滤后保存
tcpdump -i eth0 -r raw.pcap -w filtered.pcap port 80
```

### 快速参考卡片
```bash
# 最常用的组合
tcpdump -i eth0 -nn -v host 1.2.3.4          # 分析某主机
tcpdump -i eth0 -nn port 80 -A               # 看 HTTP 内容
tcpdump -i eth0 -w out.pcap -C 100 -W 5      # 滚动保存
tcpdump -r file.pcap -nn 'port 443'          # 分析已保存文件
tcpdump -i any -nn not port 22               # 排除 SSH 干扰
```

## 与Wireshark配合
```bash
tcpdump 抓包 → 保存 pcap → Wireshark 分析

tcpdump -i eth0 -w capture.pcap    # 服务器上抓包
scp user@server:capture.pcap .     # 下载到本地
# 用 Wireshark 打开 capture.pcap   # 图形化分析
```