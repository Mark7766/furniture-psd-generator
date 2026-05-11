# furniture-psd-generator

家具投标插图生成工作流 Hermes Skill

用 gpt-image-1.5 分层生图（原生透明背景）+ PIL 布局合成 + psd-tools 打包为 Photoshop 可直接编辑的 PSD 文件。

## 安装

```bash
hermes skills install https://github.com/Mark7766/furniture-psd-generator
```

## 功能

- 调用 302.ai gpt-image-1.5 生成透明背景 PNG（无需抠图）
- PIL 代码控制各图层的缩放比例和位置
- psd-tools 打包为多图层 PSD，PS 直接打开可编辑

## 依赖

```bash
pip install psd-tools Pillow requests
```

需要 302.ai API Key：https://302.ai
