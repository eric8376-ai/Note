## 参考地址
https://github.com/VoltAgent/awesome-openclaw-skills
https://github.com/clawdbot-ai/awesome-openclaw-skills-zh

https://github.com/hesamsheikh/awesome-openclaw-usecases



你有安装clawhub吗？

回复 李臻: 

## 你有安装clawhub吗？

让我检查一下：

没有安装。ClawHub 技能目前状态是 missing（需要安装 CLI）。  
  
ClawHub 是一个技能市场，可以搜索、安装和发布 AI 技能。目前系统显示有 53 个可用技能，但只有 7 个已就绪。  
  
要使用 ClawHub，需要先安装 CLI：

npm install -g clawhub

  
然后就可以用 npx clawhub 或 clawhub 命令来搜索和安装技能了。  
  
需要我帮你安装吗？

## 安装clawhub并安装，然后搜索一下有没有这个技能：wechat-publisher

回复 李臻: 

安装clawhub并安装，然后搜索一下有没有这个技能：wechat-publisher

ClawHub CLI 安装成功！现在搜索技能：

找到了！✅  
  
搜索结果：

|技能|版本|描述|相关度|
|---|---|---|---|
|wechat-publisher|v0.1.0|Wechat Publisher|⭐ 3.495|
|wechat-mp-publisher|v2.0.2|微信公众号发布|3.409|
|wechat-multi-publisher|v1.0.0|多公众号发布|3.273|
|xiaohongshu-publisher|v1.0.0|小红书自动发布助手|0.841|

wechat-publisher 存在，需要我帮你安装吗？

## 前三个哪个功能更好

三个技能对比：

|技能|版本|主要特点|适合场景|
|---|---|---|---|
|wechat-mp-publisher|v2.0.2 ⭐|远程发布 + HTTP MCP 解决 IP 变动 + 大文件上传 + 多主题排版|家用宽带/动态IP环境|
|wechat-multi-publisher|v1.0.0|单次 API 发布多篇 Markdown，支持组合草稿（主+子文章）|批量发布多篇文章|
|wechat-publisher|v0.1.0|基于 wenyan-cli，多主题 + 代码高亮 + 图片自动上传|基础单篇发布|

我的建议：  

- 🥇 wechat-mp-publisher — 版本最高(2.0.2)，迭代最成熟，功能最全面。如果你是家用宽带发布（IP 会变），这个最适合
    

- 🥈 wechat-multi-publisher — 如果你需要一次性发多篇组合文章，选这个
    

- 🥉 wechat-publisher — 功能相对基础，但代码高亮支持好
    

需要我帮你安装哪个？

回复 李臻: 

前三个哪个功能更好

⚠️ 🛠️ Exec: clawhub info wechat-multi-publisher failed: error: too many arguments. Expected 0 arguments but got 2.

## 安装 **wechat-mp-publisher** 吧，并帮我配置。参数：wx430bce130dabc58b 4eba11b2586cc1d7dcd52ec665207b9f。然后测试发布一篇关于心理的文章。




