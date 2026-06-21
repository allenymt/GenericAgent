# Vision API SOP

## ⚠️ 前置规则（必须遵守）

1. **先枚举窗口**：调用 vision 前必须先用 `pygetwindow` 枚举窗口标题，确认目标窗口存在且已激活到前台。窗口不存在就不要截图。
2. **🚫 禁止全屏截图**：必须先利用ljqCtrl截取窗口区域。能截局部（如标题栏）就不截整窗口，能截窗口就绝不全屏。全屏截图在任何场景下都不允许。
3. **能不用 vision 就不用**：如果窗口标题/本地 OCR（`ocr_utils.py`）能获取所需信息，就不要调用 vision API，省 token 且更可靠。Vision 是最后手段。

## 快速用法

```python
from vision_api import ask_vision
result = ask_vision(image, prompt="描述图片内容", timeout=60, max_pixels=1_440_000)
# image: 文件路径(str/Path) 或 PIL Image
# backend: 'claude'(默认) | 'openai' | 'modelscope'
# 返回 str：成功为模型回复，失败为 'Error: ...'
```

## ⚠️ 关键避坑（已验证踩过）

1. **vision模型必须用 mimo-v2-omni**：native_oai_mimo_config 默认的 mimo-v2.5-pro 不支持图片输入（404）。只有 `mimo-v2-omni` 支持 vision。调用时需用 code_run 手动指定 model，或修改 vision_api.py 的 VISION_MODEL_OVERRIDE。
2. **apibase 已含 /v1，URL 不能重复拼接**：vision_api.py 的 `_call_openai_compat` 拼接 URL 时，必须先 `removesuffix('/v1')` 再拼 `/v1/chat/completions`，否则 404。
3. **参数名是 image_input（位置参数）**：SOP 写的 `ask_vision(image, ...)` 是错的，实际签名是 `ask_vision(image_input, prompt, timeout, max_pixels, backend)`，第一个参数是位置参数 `image_input`。

## 如果没有 `vision_api.py`，初次构建vision能力

1. 复制 `memory/vision_api.template.py` → `memory/vision_api.py`
2. 只改头部"用户配置区"：去 `mykey.py` 里扫描变量名（⚠️ 只看名字，禁止输出 apikey 值），尝试找能用配置名填入 `CLAUDE_CONFIG_KEY` / `OPENAI_CONFIG_KEY`，`DEFAULT_BACKEND` 选后端，并测试
3. 保底：没有可用 config 时去 `https://modelscope.cn/my/myaccesstoken` 申请 token 填入 `MODELSCOPE_API_KEY`
