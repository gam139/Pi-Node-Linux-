Pi Node Linux 安装失败排查技术笔记
python


复制
from datetime import datetime

markdown = f"""# Pi Node Linux 安装失败排查：APT源配置 + 403错误分析 + 手动安装建议

> 更新时间：{datetime.now().strftime('%Y-%m-%d %H:%M')}

---

## 一、APT 环境准备与代理打通

为了让 Linux 节点能够访问 Pi Network 的软件源，需要配置 HTTP 代理并确保网络连通。

### 安装基础工具
```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg docker.io
配置 v2rayN 局域网代理
在 Windows 上运行 v2rayN，修改 config.json 文件中的监听地址：

json


复制
"inbounds": [
  {
    "tag": "socks3",
    "port": 10810,
    "listen": "0.0.0.0",
    "protocol": "mixed",
    ...
  }
]
确保 Windows 防火墙放行 10810 端口。

验证代理连通性
在 Linux 上执行：

bash


复制
nc -vz 192.168.31.42 10810
curl -x http://192.168.31.42:10810 https://apt.minepi.com/repository.gpg.key
设置 APT 代理
bash


复制
echo 'Acquire::http::Proxy "http://192.168.31.42:10810";' | sudo tee /etc/apt/apt.conf.d/99proxy
二、软件源与密钥配置
下载并部署 GPG 密钥
bash


复制
curl -x http://192.168.31.42:10810 https://apt.minepi.com/repository.gpg.key | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/pi.gpg > /dev/null
添加软件源
bash


复制
echo "deb [arch=amd64] https://apt.minepi.com/ stable main" | sudo tee /etc/apt/sources.list.d/pi.list
三、安装失败排查
安装命令
bash


复制
sudo apt-get update
sudo apt-get install pi-node
常见错误
错误命令	报错信息
apt-get install pi-node	无法定位软件包
apt-cache policy pi-node	未显示任何候选版本
curl https://apt.minepi.com/	返回 403 Forbidden
GitHub Releases 页面	无 .deb 安装包
wget https://apt.minepi.com/pool/...	下载失败
四、当前状态总结
项目	状态
v2rayN 局域网代理	✅ 已打通
APT 代理配置	✅ 已生效
GPG 密钥部署	✅ 已完成
软件源配置	✅ 已添加
索引拉取	❌ 403 Forbidden
.deb 包获取	❌ 未发布
五、后续建议
✅ 等待官方恢复 apt.minepi.com 仓库访问权限

✅ 关注 GitHub Releases 页面是否发布 .deb 安装包

✅ 将本排查过程发布到 GitHub Pages 技术博客，帮助其他节点部署者少踩坑
