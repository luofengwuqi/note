```
Agent:执行主体
Skill:能力说明
Workflow:步骤编排
Task:具体任务
```

```
感知层
认知层
执行层
记忆层
治理层
```
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
## openclaw
全局代理 在学习
### 文件结构
根目录（自动注入，每次会话必读）
| 文件 | 作用 | 类比 |
| --- | --- | --- |
| `SOUL.md` | 灵魂：我是谁、什么性格、怎么说话 | 性格设定 |
| `USER.md` | 用户档案：用户是谁、学什么、在意什么 | 老板档案 |
| `AGENTS.md` | 行为规则：怎么干活、记忆怎么记、群聊怎么发言、安全边界 | 员工手册 |
| `TOOLS.md` | 工具笔记：本地工具的使用备忘（存档路径、编码坑等） | 操作手册 |
| `IDENTITY.md` | 身份卡：名字、emoji、头像等元数据 | 工牌 |
| `HEARTBEAT.md` | 值班表：定时检查要做的事（空着 = 不主动打扰） | 值班表 |
### 常用命令
#### CLI
```bash
openclaw-cn tui #打开openclaw 控制台 很多gui没有的命令 在这里可以执行 Terminal User Interface

openclaw-cn agents add  #进入agent创建向导
```
#### TUI
```
# 日常
/status 看当前 agent、会话、模型、token 用量

/new  创建新会话 开新会话 = 翻新篇章，旧的上下文"存档"了（文件在，随时能读）
      想找回旧内容：直接打开旧 JSONL 文件看，或者说"读一下存档"——帮你把内容捞回来

/usage ( tokens | full | cost ) 显示每次回复的 token/费用


# 切换
agent <id>    │ 切换 agent（/agents 列出） 
session <key> │ 切换会话（/sessions 列出）

# 调试
 /think off/low/high │ 控制思考级别          
 /reasoning on/off   │ 显示/隐藏推理过程     
 /verbose on/off     │ 显示/隐藏工具调用细节 
 /context list       │ 看当前上下文里塞了啥  
```
## opencode
### 常用命令
```
/connent 连接大模型
/models 选择大模型
/skills 选择特技
exit 退出
```
### 创建agent
在 opencode.json(c) 配置文件中配置代理
```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "agent": {
    "build": {
      //两个选项: primary, subagent 前者可以和用户直接交互 后者只能只能被前者调用
      "mode": "primary",
      //指定模型
      "model": "deepseek/deepseek-v4-flash",
      //角色
      "prompt": "你是专业的代码构建助手，负责生成、编写、修改代码，遵循最佳实践",
      //温度:选填 0-1 0.1-0.2严谨 0.3-0.5标准 0.6-1创造力
      "temperature": 0.7,
      "tools": {
        "write": true,//写权限
        "edit": true,//
        "bash": true
      }
    },
    //在这里写新的agent
  }
}
```
# prompt
## C-R-E-A-T-E框架
```
C-Context(背景设定)
R-Requset(核心任务)
E-Explanation(流程拆解)
A-Action/Tools(工具约束)
T-Tone(语气设定)
E-Extra(边界)
```