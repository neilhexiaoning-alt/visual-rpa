---
name: visual-rpa
description: "视觉RPA桌面自动化。当用户要求操作桌面应用时使用此技能：点击图标、打开应用、在输入框输入文字、点击按钮、滚动页面等。通过截屏+视觉模型实现纯视觉定位，不依赖DOM或辅助功能API。"
metadata:
  {
    "openclaw":
      {
        "emoji": "🖱️",
        "os": ["win32"],
        "requires": { "anyBins": ["python", "python3"], "env": ["DASHSCOPE_API_KEY"] }
      }
  }
---

# 视觉 RPA 桌面自动化

> **全自动执行，各步骤之间不要等待用户确认。**

通过屏幕截图 + 通义千问视觉模型 (Qwen-VL) 实现纯视觉桌面自动化操作。

## 工作原理

1. 截取屏幕 → 缩略图粗定位目标元素
2. 全分辨率裁剪 → 精确定位坐标
3. 执行鼠标/键盘操作 → 截图验证是否成功
4. 支持复合指令自动分解为多步执行

## 使用方法

### 批量任务模式（推荐）

用 exec 工具执行命令。脚本路径固定为 `$env:TAXBOT_ROOT/skills/visual-rpa/scripts/visual_rpa.py`。

**单步任务：**

```
python "$env:TAXBOT_ROOT/skills/visual-rpa/scripts/visual_rpa.py" --mode task --task "点击打开微信"
```

**复合任务（自动分解为多步）：**

```
python "$env:TAXBOT_ROOT/skills/visual-rpa/scripts/visual_rpa.py" --mode task --task "打开微信，并打开文件传输助手聊天，在输入框输入你好，并点击发送"
```

**多步任务（手动指定每步）：**

```
python "$env:TAXBOT_ROOT/skills/visual-rpa/scripts/visual_rpa.py" --mode task --task "点击打开Chrome浏览器" "在地址栏输入 baidu.com 并回车" "在搜索框输入天气预报" "点击搜索按钮"
```

**跳过验证（更快，省 API 调用）：**

```
python "$env:TAXBOT_ROOT/skills/visual-rpa/scripts/visual_rpa.py" --mode task --no-verify --task "点击打开计算器"
```

### 参数说明

| 参数 | 说明 |
|------|------|
| `--mode task` | 批量任务模式（必须指定） |
| `--mode interactive` | 交互模式（默认） |
| `--task "指令1" "指令2"` | 要执行的操作指令，支持多个 |
| `--no-verify` | 跳过操作后的验证步骤 |
| `--model MODEL` | 视觉模型名称（默认 qwen-vl-max-latest） |
| `--api-key KEY` | API Key（默认读取 DASHSCOPE_API_KEY 环境变量） |

## 支持的操作类型

| 操作 | 指令示例 |
|------|----------|
| 点击 | "点击开始菜单"、"点击Chrome图标" |
| 双击 | "双击桌面上的回收站" |
| 右键 | "右键点击桌面空白处" |
| 输入文字 | "在搜索框输入天气预报"、"在输入框输入你好" |
| 快捷键 | "按下 Ctrl+C" |
| 滚动 | "向下滚动页面" |
| 等待 | "等待页面加载" |

## 指令编写要点

- **具体明确**：说清楚点哪个元素。"点击微信图标"比"打开微信"更精确
- **位置描述**：可以加位置提示。"点击任务栏上的微信图标"、"点击左侧列表中的文件传输助手"
- **一步一操作**：复杂操作拆成多步。也可以写复合指令，系统会自动分解
- **输入文字时**：明确说"在XXX中输入YYY"，系统会自动识别为输入操作

## 输出解读

脚本输出包含每步的执行状态：

```
  [OK] 步骤0: 点击打开微信
       click @ (375,1591)
  [OK] 步骤1: 在微信中点击文件传输助手进入聊天
       click @ (154,97)
  [FAIL] 步骤2: 在输入框中输入你好
       type @ (300,1364)
  2/3 成功
```

- **OK** = 操作成功并通过验证
- **FAIL** = 操作失败或验证未通过，默认会自动重试最多 3 次

## 常见场景

### 发送微信消息

```
python "$env:TAXBOT_ROOT/skills/visual-rpa/scripts/visual_rpa.py" --mode task --task "打开微信，打开文件传输助手聊天，在输入框输入你好，点击发送"
```

### 打开应用并操作

```
python "$env:TAXBOT_ROOT/skills/visual-rpa/scripts/visual_rpa.py" --mode task --task "点击打开Chrome浏览器" "在地址栏输入 https://www.baidu.com 并回车"
```

### 桌面操作

```
python "$env:TAXBOT_ROOT/skills/visual-rpa/scripts/visual_rpa.py" --mode task --task "右键点击桌面空白处" "点击新建文件夹"
```

## 注意事项

- 每步操作需 3-8 秒（截图 + API 调用 + 验证），不适合毫秒级响应场景
- 中文输入通过剪贴板粘贴实现，会覆盖当前剪贴板内容
- 仅操作主屏幕
- 日志和截图保存在 `./rpa_logs/` 目录，可用于调试
- 如果操作失败，检查 `./rpa_logs/rpa.log` 和截图文件排查原因
