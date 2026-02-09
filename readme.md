🚀 OpenClaw 小白保姆级教程：接入第三方 OpenAI 模型
适用版本：OpenClaw 2026.2.6 及以上 难度：⭐ (只改一个文件) 目标：把原本只支持官方 API 的 OpenClaw，改成可以使用任意“第三方/中转”服务（更便宜、更多模型）。

🤔 原理图解：我们要改什么？
在修改代码前，先看懂这张图，你就知道我们在做什么了：

代码段
graph LR
    A[你是用户] -->|提问| B(OpenClaw 主程序)
    B -->|❌ 原路| C[官方 OpenAI/Claude (太贵/连不上)]
    B -->|✅ 新路| D[第三方中转平台]
    D -->|转发| E[Claude-4.5 / GPT-4o 等模型]
    
    style B fill:#f9f,stroke:#333,stroke-width:2px
    style D fill:#bbf,stroke:#333,stroke-width:2px
我们只需要做一件事：告诉 OpenClaw，“新路”在哪里。

🛠️ 第一步：准备工作
在开始之前，请确保你手头有这三样东西（去你的中转平台后台找）：

接口地址 (Base URL)：例如 https://api.zhongzhuan.win/v1 (必须以 /v1 结尾)

API 密钥 (Key)：例如 sk-abc123456...

模型 ID：例如 claude-opus-4-5-20251101-thinking

💾 第二步：备份配置文件 (救命药)
一定要备份！万一改坏了，一行命令就能恢复。 打开终端 (Terminal)，输入：

Bash
cd ~/.openclaw
# 复制一份作为备份
cp openclaw.json openclaw.json.bak
💡 如果不小心改坏了怎么办？ 运行 cp openclaw.json.bak openclaw.json 即可恢复原样。

✍️ 第三步：修改核心配置 (重点)
我们要编辑 ~/.openclaw/openclaw.json 文件。推荐使用 VS Code 或 Subline Text 编辑，防止格式错误。

我们需要修改 三个位置。请仔细对照下面的代码。

1. 设置 API Key (位置：文件最上方)
找到 "env": { ... } 区域，把你的 Key 放进去。这样安全且方便管理。

代码段
{
  // ... 其他原有配置 ...

  "env": {
    // 👇【修改点 1】在这里填入你的 Key，名字自己起，比如 MY_API_KEY
    "MY_API_KEY": "sk-xxxxxxxxxxxxxxxxxxxxxxxx"
  },

  // ...
}
2. 注册中转商 (位置：models -> providers)
找到 "models": { ... } 区域。我们要添加一个新的 providers。

⚠️ 注意：JSON 格式非常严格，大括号 { } 要成对，上一项结束如果后面还有内容，记得加逗号 ,。

代码段
  "models": {
    "mode": "merge", 
    "providers": {
      
      // 👇【修改点 2】新增这段 "zhongzhuan" 配置（名字可自定义）
      "zhongzhuan": {
        // 填你的中转地址，记得带 /v1
        "baseUrl": "https://www.zhongzhuan.win/v1",
        
        // 这里引用上面 env 里设置的 Key，格式是 ${变量名}
        "apiKey": "${MY_API_KEY}",
        
        // 固定写法，表示兼容 OpenAI 协议
        "api": "openai-completions",
        
        // 这里列出你想用的模型
        "models": [
          {
            // 模型 ID (必须和中转平台显示的一模一样)
            "id": "claude-opus-4-5-20251101-thinking",
            // 给自己看的显示名称
            "name": "Claude 4.5 Thinking (中转)",
            // 文本输入选 text，有画图功能加 image
            "input": ["text"],
            // 下面这些参数如果不确定，可以照抄，影响不大
            "contextWindow": 200000,
            "maxTokens": 8192
          },
          {
             // 你可以按上面的格式继续添加第二个模型...
             "id": "claude-opus-4-6-20260203",
             "name": "Claude 4.6 (备用)",
             "input": ["text"],
             "contextWindow": 200000,
             "maxTokens": 8192
          }
        ]
      }
      // 👆 新增结束
      
    }
  },
3. 设置默认模型 (位置：agents -> defaults)
告诉 OpenClaw 默认使用刚才添加的模型。

代码段
  "agents": {
    "defaults": {
      "model": {
        // 👇【修改点 3】设置主模型
        // 格式必须是： "provider名字/模型ID"
        "primary": "zhongzhuan/claude-opus-4-5-20251101-thinking"
      },
      
      "models": {
        // 👇 这里设置别名，方便在日志里看
        "zhongzhuan/claude-opus-4-5-20251101-thinking": {
          "alias": "Claude 4.5 Pro"
        }
      },
      
      // ... 其他原有配置保持不变 ...
      "workspace": "..." 
    }
  }
}
🔄 第四步：重启生效
修改完配置后，必须重启 OpenClaw 的后台服务才能生效。在终端执行：

Bash
# 强制重启网关服务
launchctl kickstart -k gui/$(id -u)/ai.openclaw.gateway
然后检查状态：

Bash
openclaw status --deep
👀 看哪里？ 如果你在输出里看到了 default claude-opus-4-5...，恭喜你，配置成功了！🎉

✅ 第五步：测试一下
打开 OpenClaw 的网页端（通常是 http://127.0.0.1:18789/），新建一个对话，问它：

“你是哪个模型？请复述你的具体版本号。”

如果它回复了你配置的 Claude 4.5 或 4.6，说明一切正常！