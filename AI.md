# skills
- 将一次有价值有标准化流程的思考整成一个可复用的东西  
- 渐进式披露:按需加载  
## skill网站
- anthropic官方仓库：https://github.com/anthropics/skills
- Skills社群网站：https://skillsmp.com/zh
- 优秀开源集合：https://github.com/ComposioHQ/awesome-claude-skills
- 视频制作skill：https://github.com/remotion-dev/skills
- youtube视频剪辑skill：https://github.com/op7418/Youtube-clipper-skill
- 大师帮你创建skill的skill：https://github.com/GBSOSS/skill-from-masters
- notebookLM skill：https://github.com/PleasePrompto/notebooklm-skill
- markdown发布到X skill：https://github.com/wshuyi/x-article-publisher-skill
- AI 视频产品 Vidu Skills：https://www.vidu.cn/
## skill格式
- 文件目录
|___SKILL.md(必须有)  
|  
|__assets资源文件夹  
|  
|__references参考文件夹  
|  
|__scripts脚本文件夹

- SKILL.md咋写?
```
//元数据(yaml格式)
---
name: skill的名字
description: skill的简介 描述
---
//指令
这里可以写的比较自由 写skill执行流程
```

## 全局skill
- 在macOS 路径  
~/.config/opencode/skills  
- Windows 路径  
C:\Users\你的用户名\.config\opencode\skills  
- 如果没有全局配置skill 那么在项目使用时需要在项目根目录创建如下的路径:  
.opencode/skills/skill本体名字/skill本体

# agent
## opencode
### 常用命令
```
/models 选择大模型
/skills 选择特技
exit 退出
```