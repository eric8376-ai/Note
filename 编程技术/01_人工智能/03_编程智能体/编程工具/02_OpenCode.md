https://opencode.ai/docs

## 安装open-code

npm uninstall -g opencode-ai
npm cache clean --force
切换到国际源（否则会报错）
npm config set registry https://registry.npmjs.org
安装
npm install -g opencode-ai@latest


如果有bun的问题，执行一下语句。
npm cache clean --force  
npm uninstall -g opencode-windows-x64  
npm install -g opencode-windows-x64@latest

看起来问题仍然存在，并且很明显是 **CCcLHShield64.dll** 这个文件导致的冲突。这很可能是一个安全软件（可能是联想电脑管家或其他杀毒软件）与 Bun 不兼容。

https://linux.do/t/topic/1439614
## 配置文件
1. **Remote config** (from `.well-known/opencode`) - organizational defaults
2. **Global config** (`~/.config/opencode/opencode.json`) - user preferences
3. **Custom config** (`OPENCODE_CONFIG` env var) - custom overrides
4. **Project config** (`opencode.json` in project) - project-specific settings
5. **`.opencode` directories** - agents, commands, plugins
6. **Inline config** (`OPENCODE_CONFIG_CONTENT` env var) - runtime overrides

## 插件Oh My OpenCode

https://github.com/code-yeongyu/oh-my-opencode
https://github.com/code-yeongyu/oh-my-opencode/blob/dev/README.zh-cn.md
让LLM自己安装
获取安装指南并按照说明操作：

```shell
curl -s https://raw.githubusercontent.com/code-yeongyu/oh-my-opencode/refs/heads/master/docs/guide/installation.md
```
