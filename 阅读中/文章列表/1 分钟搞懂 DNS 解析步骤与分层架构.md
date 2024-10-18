---
source: https://mp.weixin.qq.com/s/THgHvhQh7VCt22RWdUI0Hg
create: 2024-09-02 13:53
read: false
---
# 架构师之路：必备技能，1 张大图，1 分钟搞懂 DNS 解析步骤与分层架构（无水印）

💻 DNS 解析步骤  
1️⃣ 查本地缓存  
2️⃣ 查递归（Recursive）DNS 服务器  
3️⃣ 查根（Root）DNS 服务器  
4️⃣ 查顶级（Top-level）DNS 服务器  
5️⃣ 查权威（Authoritative）DNS 服务器  
💻本地缓存（Local caches）  
1️⃣ 浏览器缓存（Browser cache）  
2️⃣ DNS 缓存（DNS cache）  
3️⃣ Hosts 文件（Hosts file）  
💻 递归（Recursive）DNS 服务器  
✅ 一般由 ISP 提供  
✅ 不存储实际域名和 IP  
✅ 存储历史查询映射结果  
💻 根（Root）DNS 服务器  
✅ 顶级域名服务器  
✅ .com .org .edu 等  
✅ 全球共 13 个  
✅ 不存储实际域名和 IP  
✅ 存储历史查询映射结果  
💻 顶级（Top-level）DNS 服务器  
✅ 管理特定二级域名  
✅ google.com xyz.org standford.edu 等  
✅ 不存储实际域名和 IP  
✅ 存储历史查询映射结果  
💻 权威（Authoritative）DNS 服务器  
✅ 管理实际域名与 IP 的映射关系  
✅ www.google.com cs.standford.edu 等  
👤作者：Kamran Ahmed  
🏠Twitter：kamranahmedse  
💪提醒：大图无水印，可直接下载。  
🎁欢迎👍+🔗+🔖  
⚡️持续发布架构师之路上的必备技能！