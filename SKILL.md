---
name: furniture-psd-generator
description: "家具投标插图生成工作流：用 gpt-image-1.5 分层生图（原生透明背景）+ PIL 布局合成 + psd-tools 打包为 PS 可用 PSD。"
version: 1.0.0
metadata:
  hermes:
    tags: [psd, furniture, ai-image, gpt-image, photoshop, 家具, 投标]
---

# 家具投标插图 PSD 生成工作流

## 适用场景

为家具行业投标员生成多图层 PSD 插图。每层用 AI 独立生图，代码控制布局比例，最终打包为 Photoshop 可直接编辑的 `.psd` 文件。

## 核心原理

> 分别用 gpt-image-1.5 生成透明背景 PNG → PIL 缩放定位到画布 → psd-tools 打包为 PSD

- **gpt-image-1.5**：原生支持 `background: transparent`，直接输出 RGBA PNG，无需抠图
- **gpt-image-2**：❌ 302.ai 中转不支持透明背景参数（返回 503 参数错误）
- **gpt-image-1**：✅ 支持透明背景，但价格更高
- **rembg 抠图**：效果差，室内家具场景边缘不干净，不推荐

## 环境依赖

```bash
pip install psd-tools Pillow requests
```

## API 配置

- **提供商**：302.ai（国内可直连）
- **接口**：`https://api.302.ai/v1/images/generations`
- **模型**：`gpt-image-1.5`
- **关键参数**：
  ```json
  {
    "model": "gpt-image-1.5",
    "background": "transparent",
    "output_format": "png",
    "response_format": "b64_json",
    "size": "1024x1024"
  }
  ```
- **代理**（中国境内需要）：设置环境变量 `HTTPS_PROXY=http://your-proxy:port`，代码会自动读取

## 标准工作流（3步）

### 第1步：生成各图层透明 PNG

```python
import requests, base64, time, os
from PIL import Image
from io import BytesIO

KEY = 'YOUR_302AI_KEY'
# 代理：从环境变量读取，不要硬编码
import os
proxy = os.environ.get('HTTPS_PROXY') or os.environ.get('https_proxy')
PROXIES = {'https': proxy, 'http': proxy} if proxy else None
DIR = '/tmp/furniture_psd'
os.makedirs(DIR, exist_ok=True)

layers = [
    ('layer_bg.png',   'modern luxury furniture showroom interior, empty room, wooden floor, warm lighting, wide angle, no furniture'),
    ('layer_sofa.png', 'a modern luxury beige sofa set, isolated on transparent background, no floor, no shadow, product shot'),
    ('layer_text.png', 'elegant brand label text "HOME LUXURY" in gold font, transparent background, minimal design'),
]

images = {}
for fname, prompt in layers:
    print(f'生成 {fname}...')
    r = requests.post('https://api.302.ai/v1/images/generations',
        headers={'Authorization': f'Bearer {KEY}', 'Content-Type': 'application/json'},
        json={
            'model': 'gpt-image-1.5',
            'prompt': prompt,
            'n': 1,
            'size': '1024x1024',
            'background': 'transparent',
            'output_format': 'png',
            'response_format': 'b64_json',
        },
        proxies=PROXIES, timeout=120)
    img = Image.open(BytesIO(base64.b64decode(r.json()['data'][0]['b64_json']))).convert('RGBA')
    img.save(f'{DIR}/{fname}')
    images[fname] = img
    print(f'  ✅ mode={img.mode}')
    time.sleep(2)  # 避免限流
```

### 第2步：PIL 布局合成预览图

```python
SIZE = (1024, 1024)

bg   = images['layer_bg.png'].resize(SIZE)
sofa = images['layer_sofa.png']
text = images['layer_text.png']

# 沙发：70% 宽，下方居中
sofa_w = int(SIZE[0] * 0.70)
sofa_h = int(sofa.height * sofa_w / sofa.width)
sofa = sofa.resize((sofa_w, sofa_h), Image.LANCZOS)
sofa_x = (SIZE[0] - sofa_w) // 2
sofa_y = SIZE[1] - sofa_h - 60

# 文字：45% 宽，右下角
text_w = int(SIZE[0] * 0.45)
text_h = int(text.height * text_w / text.width)
text = text.resize((text_w, text_h), Image.LANCZOS)
text_x = SIZE[0] - text_w - 40
text_y = SIZE[1] - text_h - 40

# 合成预览
canvas = Image.new('RGBA', SIZE, (255, 255, 255, 255))
canvas.alpha_composite(bg)
canvas.alpha_composite(sofa, (sofa_x, sofa_y))
canvas.alpha_composite(text, (text_x, text_y))
canvas.convert('RGB').save(f'{DIR}/composite.png')
print('✅ composite.png — 先给用户看预览，确认比例后再生成 PSD')
```

### 第3步：生成 PSD

```python
from psd_tools import PSDImage

def place_on_canvas(img, x, y, size=SIZE):
    """把裁剪过的图层放回全尺寸画布，保留位置信息"""
    canvas = Image.new('RGBA', size, (0, 0, 0, 0))
    canvas.alpha_composite(img, (x, y))
    return canvas

psd = PSDImage.new('RGBA', SIZE)
psd.create_pixel_layer(bg, name='Background')
psd.create_pixel_layer(place_on_canvas(sofa, sofa_x, sofa_y), name='Sofa')
psd.create_pixel_layer(place_on_canvas(text, text_x, text_y), name='BrandText')
psd.save(f'{DIR}/furniture.psd')
print(f'✅ furniture.psd {os.path.getsize(f"{DIR}/furniture.psd")//1024}KB')
```

## 布局参数调节

用户觉得比例不对时，只改这几个数字，**不需要重新调用 API**：

| 参数 | 含义 | 默认值 | 调节范围 |
|------|------|--------|---------|
| `0.70` | 沙发占画面宽度比例 | 70% | 40%-85% |
| `sofa_y` | 沙发距底部距离 | 60px | 0-200px |
| `0.45` | 文字占画面宽度比例 | 45% | 20%-60% |
| `text_x/text_y` | 文字位置 | 右下角 | 自由调整 |

## Skill 安装方式

已发布到 GitHub：
```bash
hermes skills install https://github.com/Mark7766/furniture-psd-generator
```

> **注意**：`hermes skills publish --repo` 命令需要 GitHub token 有 fork 权限，通常会报错。
> 正确方式是手动 git push：
> ```bash
> git clone https://github.com/YOUR/repo.git
> cp SKILL.md repo/ && cd repo
> git add . && git commit -m "feat: add skill"
> GIT_SSL_NO_VERIFY=1 git push https://TOKEN@github.com/YOUR/repo.git main
> ```

---

## 常见问题

### gpt-image-2 透明背景 503 错误
302.ai 中转的 gpt-image-2 不支持 `background: transparent` 参数，换用 `gpt-image-1.5` 即可。

### psd-tools create_pixel_layer 图层为空
psd-tools 在某些 Python 版本下写入像素数据有 bug。确保图层是全尺寸 `SIZE` 的 RGBA Image，用 `place_on_canvas()` 把缩放后的图层放回全尺寸画布再写入。

### 各层比例失调（沙发占满整张图）
AI 独立生图时每层都以"主角"视角构图，合成后必然失调。**必须用代码缩放定位**，不能直接叠加原始图层。

### 合成图白色背景变黑
`Image.new('RGBA', SIZE, (0,0,0,0))` 合成时底层透明会显示黑色，改为 `(255,255,255,255)` 白底。

## 扩展图层建议

可以增加更多图层丰富画面：
- `layer_chair.png` — 单椅（缩放到30%，放沙发旁边）
- `layer_plant.png` — 绿植装饰（放角落）
- `layer_rug.png` — 地毯（放沙发下方）

每加一层只需一次 API 调用，PIL 代码控制位置。
