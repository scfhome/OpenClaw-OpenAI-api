## 🚀 OpenClaw 小白保姆级教程：接入第三方 OpenAI 模型

> **适用版本**：2026.2.6 及之后的 OpenClaw（参考仓库 `openclaw/openclaw`）  
> **难度**：⭐（只需修改一个配置文件）  
> **场景**：你有一个“第三方接口”（比如中转服务），想让 OpenClaw 默认用它的模型（如 Claude 4.5、GPT-4o 等），而不是官方昂贵的 API。

---

### 📚 一、这篇教程会帮你做到什么？

按照本教程，你会完成：

- **接入第三方 OpenAI 兼容接口** 到 OpenClaw
- **让主路由默认使用第三方的某个模型**
  - 例如：`claude-opus-4-5-20251101-thinking`
- **全流程只改一个文件**：`~/.openclaw/openclaw.json`

---

### 🧐 原理图解

从“原来直连官方”，变成“走你的中转平台”：

```text
你（用户） ──> OpenClaw 主程序
     ├─（❌ 原路：官方 OpenAI，太贵/可能连不上）
     └─（✅ 新路：你自己的第三方中转平台）
                    ↓
          Claude 4.5 / GPT-4o / 其他模型
```

- **我们要做的事情**：  
  告诉 OpenClaw：“别走官方了，走我配置的这个 `baseUrl + apiKey + modelId`。”

---

### 🛠️ 二、你要提前准备的三样东西

请到你的中转平台 / API 代理商后台，准备好下面 3 个信息：

- **API Base URL（接口地址）**
  - 示例：`https://api.zhongzhuan.win/v1`
  - ⚠️ **重要：最后必须带 `/v1`**

- **API Key（密钥）**
  - 示例：`sk-abc123456...`
  - 这一串就是你的钱，一定不要泄露给别人。

- **Model ID（模型 ID）**
  - 示例：`claude-opus-4-5-20251101-thinking`
  - 必须和平台提供的一字不差。

---

### 💾 三、先备份配置（出错能秒回滚，三系统通用）

这一步不能跳过！  
万一后面 JSON 改坏了，一行命令就能恢复。

不同系统推荐这样操作（**任选自己系统那一块照抄**）：

- **macOS / Linux（bash / zsh / 其他常见 shell）**：

  ```bash
  cd ~/.openclaw

  # 备份当前配置
  cp openclaw.json openclaw.json.bak

  # 如果改坏了，恢复备份
  cp openclaw.json.bak openclaw.json
  ```

- **Windows PowerShell**：

  ```powershell
  cd "$env:USERPROFILE\.openclaw"

  # 备份当前配置
  Copy-Item openclaw.json openclaw.json.bak -Force

  # 如果改坏了，恢复备份
  Copy-Item openclaw.json.bak openclaw.json -Force
  ```

- **Windows 命令行（cmd）**：

  ```cmd
  cd /d %USERPROFILE%\.openclaw

  rem 备份当前配置
  copy /Y openclaw.json openclaw.json.bak

  rem 如果改坏了，恢复备份
  copy /Y openclaw.json.bak openclaw.json
  ```

不管哪个系统，只要 `.bak` 备份在，就能**瞬间复原**。

---

### ✍️ 四、修改核心配置（只需改 3 处）

请用 VS Code、Sublime Text 或 Cursor 编辑 `openclaw.json`，不同系统路径对应关系如下：

- **macOS / Linux**：`~/.openclaw/openclaw.json`（`~` 就是你的用户主目录）
- **Windows**：`%USERPROFILE%\.openclaw\openclaw.json` 或 `C:\Users\你的用户名\.openclaw\openclaw.json`


---

#### 📍 第一处：配置 API Key（推荐写在 `env`，更安全）

在文件最上方，找到 `"env": { ... }` 这一块，如果没有就加上：

```json
  // ... 前面是 meta, wizard 等 ...

  "env": {
    // 👇【新增这一行】
    // 名字自己起（比如 MY_API_KEY），后面填你的 sk-密钥
    "MY_API_KEY": "sk-xxxxxxxxxxxxxxxxxxxxxxxx"
  },

  // ...
```

- 这里的 `"MY_API_KEY"` 只是一个名字，可以换成你喜欢的  
- 下面第二处我们会通过 `"${MY_API_KEY}"` 来引用它📍📍📍📍

---

#### 📍 第二处：注册第三方服务商（`models.providers`）

在同一个文件里，找到类似：

```json
  "models": {
    "mode": "merge",
    "providers": {
      // ...
    }
  }
```

在 `"providers": { ... }` 里面，新增你自己的 provider 配置，例如叫 `zhongzhuan`：

```json
  "models": {
    "mode": "merge",
    "providers": {

      // 👇【从这里开始新增】===
      "zhongzhuan": {
        // 1. 【必须修改】你的中转地址，记得带 /v1
        "baseUrl": "https://api.zhongzhuan.win/v1",

        // 2. 这里引用上面 env 里设置的 Key。📍📍📍📍注意⚠️是$字符+标题名字{MY_API_KEY}（你起的 sk 名字），不是 sk-abc 密钥！！⚠️
        "apiKey": "${MY_API_KEY}",

        // 3. 固定写法，表示兼容 OpenAI 协议
        "api": "openai-completions",

        // 4. 这里列出你想用的模型
        "models": [
          {
            // 【必须修改】模型 ID：和中转平台显示的一模一样
            "id": "claude-opus-4-5-20251101-thinking",

            // 给自己看的显示名称
            "name": "Claude 4.5 Thinking (中转)",

            // 文本输入选 text，有画图功能再加 "image"
            "input": ["text"],

            // 不确定就照抄这两个
            "contextWindow": 200000,
            "maxTokens": 8192
          },
          {
            // 【可选】如果有第二个模型，可以继续加...
            "id": "claude-opus-4-6-20260203",
            "name": "Claude 4.6 (备用)",
            "input": ["text"],
            "contextWindow": 200000,
            "maxTokens": 8192
          }
        ]
      }
      // 👆【新增结束】===

    }
  },
```

> **记住一个格式**：  
> 以后用的时候，模型的完整名字是：  
> `provider名字/模型ID`，例如：  
> `zhongzhuan/claude-opus-4-5-20251101-thinking`

---

#### 📍 第三处：把主路由切过去（`agents.defaults`）

在 `openclaw.json` 里找到：

```json
  "agents": {
    "defaults": {
      "model": {
        // ...
      },
      "models": {
        // ...
      },
      "workspace": "/Users/你的用户名/.openclaw/workspace"
    }
  }
```

把里面的 `"model"` 和 `"models"` 改成类似下面这样：

```json
  "agents": {
    "defaults": {
      "model": {
        // 👇【修改这一行】
        // 格式是： "provider名字/模型ID"
        "primary": "zhongzhuan/claude-opus-4-5-20251101-thinking"
      },

      "models": {
        // 👇【可选】这里设置别名，方便在日志里看
        "zhongzhuan/claude-opus-4-5-20251101-thinking": {
          "alias": "Claude 4.5 Pro"
        }
      },

      // ... 其他原有配置不要动 ...
      "workspace": "/Users/你的用户名/.openclaw/workspace"
    }
  }
```

- **关键：** `primary` 这一行就决定了“默认主路由用哪个模型”

---

### 🔄 五、重启 / 重载 OpenClaw，让配置生效

修改完 `openclaw.json` 后，需要让 OpenClaw 重新加载配置。不同系统情况略有差异：

- **macOS（官方推荐方式，有 LaunchAgent）**  
  打开终端，执行（整行复制）：

  ```bash
  # 强制重启网关服务
  launchctl kickstart -k gui/$(id -u)/ai.openclaw.gateway
  ```

- **Linux / Windows**  
  官方仓库中，安装方式和运行方式可能有多种（docker、systemd、手动运行等），对应的“重启方式”也不一样。  
  为了**绝对严谨、不乱教错误命令**，这里建议：

  - 直接打开官方文档：`https://github.com/openclaw/openclaw`
  - 根据你当初安装时使用的方式，找到对应的：
    - **Run on Linux / Run on Windows / systemd / docker compose** 等小节
  - 用“先停掉旧进程 → 再重新启动”的方式重启即可

  一般规则是：**你最开始是怎么启动的，就用同样的方式再启动一次**。

重启/重载后，用同一条命令检查状态（**三大系统通用**）：

```bash
openclaw status --deep
```

- 如果在输出里看到了类似：`default claude-opus-4-5-20251101-thinking` 这样的字样  
  → **恭喜你，配置成功了！🎉**

---

### ✅ 六、验证测试

#### 方式一：用 Dashboard 测试

1. 打开浏览器访问：

   ```text
   http://127.0.0.1:18789/
   ```

2. 新建一个对话，发一句：

   > “你是哪个模型？请复述你的具体版本号。”

3. 同时打开你的中转平台后台，看“日志”或“用量”里是否有请求产生。

只要看到有调用记录，就说明 OpenClaw 已经在走你的第三方接口了。

---

### ❓ 七、常见问题（Q&A）

- **Q1：保存时编辑器报错（JSON Error）怎么办？**  
  - 99% 是逗号问题：
    - 多写了：列表 `[]` 或对象 `{}` 的**最后一项后面不能有逗号**
    - 少写了：两项之间**必须有逗号**
  - 建议：把内容复制到在线 JSON 校验工具里检查一下。

- **Q2：我想换回原来的模型怎么办？**  
  - 找到第三处的：
    - `"primary": "zhongzhuan/claude-opus-4-5-20251101-thinking"`
  - 把它改回原来的（例如 `openai/gpt-4o`、`zai/glm-4.7` 等）
  - 再执行一次：
    ```bash
    launchctl kickstart -k gui/$(id -u)/ai.openclaw.gateway
    ```

- **Q3：为什么配置了没反应？**  
  - 检查 `baseUrl` 后面是不是**漏了 `/v1`**
  - 检查 `apiKey` 里的环境变量名是否和 `env` 里写的一模一样：
    - 上面是 `"MY_API_KEY"`，下面就必须是 `"${MY_API_KEY}"`

---

### 🧠 八、一页记完的极简心智模型

```text
第三方平台
  ├─ Base URL: https://你的域名/v1
  ├─ API Key: sk-xxxxxx
  └─ 模型 ID: claude-opus-4-5-20251101-thinking

          ↓ 写进 openclaw.json

openclaw.json
  ├─ env.MY_API_KEY                 → 存你的 Key
  ├─ models.providers.zhongzhuan    → 描述这个中转平台
  │    └─ models[]                  → 列出所有要用的模型
  └─ agents.defaults.model.primary  → 选一个当“默认主路由模型”
```

**以后你要接其他平台（CloseAI、OpenRouter、自建中转……），只需要改 3 个地方：**

1. **换 `baseUrl`**
2. **换 `apiKey` 名字和实际值**
3. **换 `models` 里的模型 `id` / `name` / 配置**

其他结构都可以直接照抄这篇“超小白快速版”教程。

