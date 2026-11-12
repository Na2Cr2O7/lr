# lr

## **L**ightweight Text **R**ecognition

用 **Windows 自带的 OCR**（`Windows.Media.Ocr`）识别文字。纯 Rust CLI，
不依赖任何第三方 OCR 库，也不需要联网。

## 用法

```powershell
#截图并读取图片上所有文字。
lr

#截图并读取图片上所有文字到txt中，为中文。
lr>o.txt -l zh-Hans-CN

# 目录：把目录里所有图片整张识别
lr screenshots\

# 单张图片
lr a.png
```

```powershell
lr -s                    # 系统装了哪些 OCR 语言
lr -j                    # 详细 JSON（每行、每个字的坐标）
lr -o name.json -j       # 写到文件
lr -l zh-Hans-CN         # 指定语言（默认跟随系统）
lr -c other.ini          # 指定 config.ini
lr --save shot.png       # 截图存到别处
lr --no-save             # 只识别，不落盘
lr --image shot.png      # 不截图，直接读现成的图（调试用）
lr --bbox 376,86,750,32  # 手动指定裁剪区域（调参用）
lr --upscale 2           # 手动指定放大倍数（1..8，调参用）
```

退出码：`0` 成功，`1` 参数错误，`2` 读配置/文件失败，`3` 有图片识别失败。

## 截图是怎么做的

不带路径时 `lr` 自己截屏。





## 两个踩过的坑（都写在代码注释里）

1. **文字贴边会让 OCR 完全失效**：同样一段字，裁成 750×32 一个字都读不出，
   上下各留 16 像素（750×64）就能全读对。所以裁剪时按高度的 1/2 加了留白
   （`pad_region`），坐标仍然换算回原图；留白带进来的"邻居文字"会按请求区域
   再过滤一遍（`intersects`），只留落在聊天名字那条里的行。
2. **`BitmapDecoder.GetSoftwareBitmapTransformedAsync` 不能用**：`Bounds` + 缩放的
   组合在这台机器上要么返回空白图、要么报 `E_INVALIDARG`，所以改成自己按像素
   裁剪 + 双线性放大（`crop_and_scale`）。


## 测试

```powershell
cargo test
```

测试已通过。
