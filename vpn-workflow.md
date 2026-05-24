# VPN 搭建完整工作流（自用 + 接单）

## 定价参考

| 服务 | 价格 | 时间 |
|------|------|------|
| 单设备安装 | ¥50-80 | 15 分钟 |
| 三设备全家桶（手机+电脑+平板） | ¥128-168 | 30 分钟 |
| 全套代购（含 VPS 代买 + 搭建 + 三设备） | ¥288-388 | 40 分钟 |

---

## 第一阶段：买 VPS

### 1.1 注册购买
- 网站：aaitr.com
- 选 **静态家宽 VPS**（最低配：1核/512M/10G SSD）
- 系统选：**Ubuntu 22.04** 或 **Debian 12**
- 地区选：美国/日本/新加坡（哪个快选哪个）

### 1.2 拿到三样东西
```
IP 地址：xxx.xxx.xxx.xxx
用户名：root
密码：  客户自己设的
```

---

## 第二阶段：FinalShell 连接

### 2.1 下载 FinalShell
- Windows/Mac 都支持
- 官网：hostbuf.com

### 2.2 连接 VPS
```
1. 点左上角 + 号 → SSH 连接
2. 名称：客户名-VPN
3. 主机：VPS IP
4. 端口：22
5. 用户名：root
6. 密码：VPS 密码
7. 点确定 → 双击连接
```

---

## 第三阶段：安装 WireGuard

### 3.1 更新系统
```bash
apt update && apt upgrade -y
```

### 3.2 一键安装
```bash
curl -L https://raw.githubusercontent.com/angristan/wireguard-install/master/wireguard-install.sh | bash
```

### 3.3 安装过程填什么
```
第一个客户端名：iphone
端口：51820（直接回车）
DNS：1.1.1.1（输 2 选 Cloudflare）
网卡：直接回车
```

---

## 第四阶段：调试 & 防火墙

### 4.1 检查是否跑起来
```bash
systemctl status wg-quick@wg0
```
看到 `active (running)` 就是成了。

### 4.2 开防火墙
```bash
ufw allow 51820/udp
ufw allow 22/tcp
ufw enable
# 输入 y 确认
```

### 4.3 查看客户端列表
```bash
wg show
```

---

## 第五阶段：iPhone 配置

### 5.1 给 iPhone 生成配置
```bash
bash wireguard-install.sh
# 选 1：Add a new user
# 客户端名：iphone
```
生成的文件在：`/root/iphone.conf`

### 5.2 把配置文件发给 iPhone
**方法一：二维码（推荐，不出镜也能操作）**
```bash
apt install qrencode -y
qrencode -t ansiutf8 < /root/iphone.conf
```
手机 WireGuard App 扫码即导入。

**方法二：临时 HTTP 下载**
```bash
cd /root
python3 -m http.server 8080 &
```
手机 Safari 打开：`http://VPS的IP:8080/iphone.conf`

### 5.3 iPhone 端操作
1. App Store 下载 **WireGuard**
2. Safari **无痕模式** 打开下载链接
3. 下载 `.conf` 文件
4. 点分享 → 用 WireGuard 打开
5. 点连接开关，看到 VPN 图标即成功

### 5.4 关掉临时服务器
VPS 上 `Ctrl+C` 关掉 `python3 -m http.server`。

---

## 第六阶段：电脑配置

```bash
bash wireguard-install.sh
# 选 1：Add a new user
# 客户端名：pc
```

生成的 `pc.conf` 复制内容 → 电脑 WireGuard 客户端 → 导入配置 → 连接。

---

## 第七阶段：验收

### iPhone 验收
- 开 VPN → 打开 Safari 访问 google.com → 能打开 = 成功
- 关 VPN → 访问不了 = 验证通过

### 电脑验收
- 开 VPN → 浏览器访问 youtube.com → 能看视频 = 成功
- 测速网站测一下：fast.com

---

## 常见问题排查

| 问题 | 解决 |
|------|------|
| 连上但不能上网 | `ufw allow 51820/udp`，重启 `systemctl restart wg-quick@wg0` |
| iPhone 连不上 | 检查 VPS 防火墙，确保 51820/UDP 开放 |
| 速度慢 | 换节点（日本/新加坡/美国测试哪个快） |
| 配置文件丢了 | VPS 上 `/root/iphone.conf` 重新发 |

---

## 视频脚本要点

**片头**：30 秒搞定 VPN，iPhone/Win/Mac 全能用
**第一阶段**：买 VPS（1 分钟）
**第二阶段**：一键安装 WireGuard（2 分钟）
**第三阶段**：iPhone 配置演示（1 分钟）
**第四阶段**：测试效果（30 秒）
**片尾**：需要代搭建 + V：xxxxx

总时长控制在 5-8 分钟，信息密度要高，废话不要多。
