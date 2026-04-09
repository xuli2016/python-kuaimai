# python-cloud-demo

`python-cloud-demo` 是发给对接用户的 Python 示例工程。

这个目录已经包含了：

- `CloudExample.py`
- `requirements.txt`
- `packages/` 里的本地 SDK wheel


- Python 交付方式：`CloudExample.py + packages/*.whl`

也就是说，对接用户拿到这个目录之后，安装依赖、改参数、运行脚本即可。

## 目录说明

- `CloudExample.py`
  示例入口
- `requirements.txt`
  安装 SDK 的依赖文件。会优先安装 `packages/` 里的本地 wheel。
- `packages/`
  随 demo 一起交付的 SDK wheel。

说明：

- 当前交付版不使用 `.env`
- 当前交付版不带 mock
- 执行 `CloudExample.py` 时会直接调用真实接口

## 运行环境要求

建议使用：

- Python `3.10+`

推荐先创建虚拟环境：

macOS / Linux:

```bash
cd python-cloud-demo
python3 -m venv .venv
source .venv/bin/activate
```

Windows:

```bash
cd python-cloud-demo
python -m venv .venv
.venv\Scripts\activate
```

## 需要安装哪些依赖

### 1. Python 依赖

执行：

```bash
pip install -r requirements.txt
```

`requirements.txt` 会安装本地 wheel，例如：

```text
./packages/python_kuaimai_core-0.1.0-py3-none-any.whl
```

然后 `pip` 会继续安装 SDK 依赖的 Python 包，例如：

- `requests`
- `Pillow`
- `qrcode`
- `reportlab`
- `treepoem`
- `pypdfium2`

如果客户机器不能直接访问 PyPI，需要提前准备：

- 公司内部 pip 镜像
- 或者所有依赖的离线 wheel 包

如果 `packages/` 目录里没有 SDK wheel，说明交付包不完整。

### 2. 系统依赖

#### Ghostscript

如果会用到条码、模板绘图等能力，建议安装 Ghostscript。

macOS:

```bash
brew install ghostscript
```

Ubuntu / Debian:

```bash
sudo apt-get update
sudo apt-get install -y ghostscript
```

Windows:

- 安装 Ghostscript
- 确保 Ghostscript 可执行文件已加入 `PATH`

如果不确定是否会用到，安装上最稳妥。

### 3. 字体

如果模板中指定了字体名称，客户机器上也需要安装对应字体，否则打印效果可能和预期不一致。

常见中文字体例如：

- `SimSun`
- `KaiTi`
- `Microsoft YaHei`

注意：

- 模板打印最依赖本机字体
- 图片直打对本机字体依赖较少
- PDF 直打通常主要取决于 PDF 自身内容，但不同机器的图像处理环境仍可能带来轻微差异

## 用户怎么使用

### 第一步：安装依赖

```bash
cd python-cloud-demo
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

### 第二步：修改 `CloudExample.py` 顶部参数

打开 `CloudExample.py`，先修改最上面的这些全局变量：

```python
APP_KEY = ""
SECRET = ""
TEST_SN = ""
```

一般至少需要填写：

- `APP_KEY`
  快麦开放平台 `appid`
- `SECRET`
  快麦开放平台 `secret`
- `TEST_SN`
  打印机序列号

其他参数按实际要跑的示例填写，例如：

- `DEVICE_KEY`
  设备绑定 / 解绑
- `TSPL_TEMPLATE_ID`
  标签模板 ID
- `ESC_TEMPLATE_ID`
  小票模板 ID
- `RENDER_DATA`
  单条模板渲染数据
- `RENDER_DATA_ARRAY`
  批量模板渲染数据
- `RESULT_JOB_ID`
  打印结果查询
- `IMAGE_PATH`
  图片打印时的本地图片路径
- `IMAGE_BASE64`
  图片打印时的 base64 内容
- `PDF_PATH`
  PDF 打印时的本地 PDF 路径
- `IMEI`
  菜鸟云打印机相关接口
- `CAINIAO_CODE`
  菜鸟绑定接口

注意：

- `IMAGE_PATH` 和 `PDF_PATH` 建议填绝对路径
- Windows 路径建议写成 `D:/demo/test.pdf` 这种形式，避免反斜杠转义问题
- 如果填了 `IMAGE_BASE64`，脚本会优先使用 `IMAGE_BASE64`

### 第三步：在 `main()` 中只放开一个示例

`CloudExample.py` 的 `main()` 里已经列好了所有示例。

建议每次只放开一个，例如：

```python
pretty_print("tsplImagePrint", tspl_image_print_example(client))
```

其他示例先保持注释状态。

### 第四步：运行脚本

```bash
python CloudExample.py
```

脚本会输出接口返回结果，例如：

```json
{
  "status": true,
  "data": {
    "jobIds": [
      "1775639212956"
    ]
  },
  "message": null,
  "code": null
}
```

## 示例函数与常见用途

`CloudExample.py` 中包含这些示例：

- `bind_device_example`
  绑定设备
- `unbind_device_example`
  解绑设备
- `query_device_status_example`
  查询设备状态
- `esc_template_print_example`
  小票模板连续纸打印
- `tspl_template_print_example`
  标签模板间隙纸打印
- `tspl_template_write_example`
  小票模板间隙纸打印
- `tspl_xml_write_example`
  TSPL 自定义 XML 打印 间隙纸打印
- `esc_xml_write_example`
  ESC 自定义 XML 打印 连续纸打印
- `result_example`
  查询打印结果
- `tspl_image_print_example`
  标签机图片直打
- `tspl_pdf_print_example`
  标签机 PDF 单页打印
- `tspl_pdfs_print_example`
  标签机 PDF 多页打印
- `esc_image_print_example`
  小票机图片直打
- `esc_pdf_print_example`
  小票机 PDF 单页打印
- `esc_pdfs_print_example`
  小票机 PDF 多页打印
- `broadcast_example`
  语音播报
- `cancel_job_example`
  取消打印任务
- `adjust_device_density_example`
  调节设备浓度
- `get_cainiao_code_example`
  获取菜鸟云打印机 code
- `cainiao_bind_example`
  绑定菜鸟云打印机
- `cainiao_print_example`
  菜鸟云打印图片打印
- `cainiao_tspl_template_print_example`
  菜鸟云打印标签模板打印

## 对接时需要注意的事项

### 1. 这是直接请求真实接口的交付版

当前 `CloudExample.py` 没有 mock，运行就是实际调用云端接口。

所以在测试前请确认：

- `APP_KEY`
- `SECRET`
- `TEST_SN`
- 模板 ID
- 图片 / PDF 路径

都已经填写正确。

### 2. 建议一次只测一个示例

不要同时放开多个 `pretty_print(...)`。

原因：

- 排查问题更简单
- 避免连续下发多个真实打印任务
- 更容易判断是哪一个接口报错

### 3. 模板打印和字体强相关

如果模板里用了指定字体，而客户机器没有安装同名字体，可能出现：

- 换行位置变化
- 文本宽度变化


### 4. 图片和 PDF 打印会自动做适配

SDK 会对图片、PDF 做适当缩放后再下发打印。

例如：

- `tsplPdfPrint` / `tsplPdfsPrint` 会按标签尺寸处理
- `escPdfPrint` / `escPdfsPrint` 会按小票纸宽处理

但如果源文件本身特别大、特别复杂，仍然可能影响打印清晰度或处理时间。

### 5. 多页打印返回的是“每页一个结果”

像下面这些多页接口：

- `tspl_pdfs_print_example`
- `esc_pdfs_print_example`

返回值里的 `data` 通常是一个列表，每一项对应一页的处理结果。

排查问题时不要只看最外层 `status`，也要看列表里每一项是否成功。

### 6. 路径必须真实存在

如果测试这些接口：

- `tspl_image_print_example`
- `esc_image_print_example`
- `cainiao_print_example`
- `tspl_pdf_print_example`
- `tspl_pdfs_print_example`
- `esc_pdf_print_example`
- `esc_pdfs_print_example`

请确认：

- `IMAGE_PATH` 指向真实图片
- `PDF_PATH` 指向真实 PDF

### 7. 菜鸟云打印接口只适用于对应设备

下面这些示例只在对应设备和绑定状态正确时才能测试：

- `get_cainiao_code_example`
- `cainiao_bind_example`
- `cainiao_print_example`
- `cainiao_tspl_template_print_example`

## 交付包更新后怎么重装

如果你拿到了新的 wheel 包，需要在当前虚拟环境里强制重装一次：

```bash
pip install --force-reinstall --no-deps ./packages/python_kuaimai_core-0.1.0-py3-none-any.whl
```

否则当前环境里可能还在使用旧版本 SDK。

## 常见问题排查

### 运行时报 `未找到 kuaimai_core`

说明还没有先执行：

```bash
pip install -r requirements.txt
```

### 运行时报认证或设备相关错误

优先检查：

- `APP_KEY`
- `SECRET`
- `TEST_SN`
- 模板 ID
- `IMEI`
- `CAINIAO_CODE`

### 打印效果和预期不一致

优先检查：

- 客户机器字体是否齐全
- Ghostscript 是否安装
- 使用的是模板打印、图片直打还是 PDF 直打
- 图片 / PDF 原文件尺寸是否过大

## 一句话总结

对接用户的使用方式很简单：

1. 安装依赖
2. 改 `CloudExample.py` 顶部参数
3. 在 `main()` 中只打开一个示例
4. 运行 `python CloudExample.py`

