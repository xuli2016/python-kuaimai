# Python SDK 说明文档

本文档面向 Python 对接方，说明 `python-cloud-demo` 中随包交付的 Python SDK 用法。示例代码和接口组织以当前 `CloudExample.py` 为准。

## 一、快速开始

### 1. 安装运行环境

建议使用 Python 3.10 及以上版本。

Linux / macOS：

```bash
cd python-cloud-demo
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

Windows：

```bash
cd python-cloud-demo
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

### 2. 申请开放平台凭证

初始化 SDK 前需要先在快麦开放平台申请：

- `appid`
- `secret`

### 3. 初始化客户端

```python
from kuaimai_core import KuaimaiClient

client = KuaimaiClient.create_client(appid, secret)
```

### 4. 构造请求对象并调用

以查询设备状态为例：

```python
import json

from kuaimai_core import KuaimaiClient
from kuaimai_core.request import QueryDeviceStatusRequest

client = KuaimaiClient.create_client(appid, secret)

request = QueryDeviceStatusRequest(
    sns=json.dumps(["KM118DW123"], ensure_ascii=False),
)

response = client.get_acs_response(request)
print(response.to_dict())
```

### 5. 参考完整示例

完整可运行示例见 `CloudExample.py`。当前交付版本不依赖 `.env`，也不包含 mock 逻辑，直接修改文件顶部参数即可运行。

## 二、通用说明

### 1. 通用响应格式

大部分接口都返回如下结构：

| 参数名 | 含义 | 类型 | 说明 |
| --- | --- | --- | --- |
| `status` | 请求是否成功 | `bool` | `True` 为成功，`False` 为失败 |
| `data` | 响应数据 | `object` | 具体结构由接口决定 |
| `message` | 错误说明 | `str` | 调用失败时一般会返回原因 |
| `code` | 错误码 | `int` | 具体错误码由开放平台返回 |

### 2. 图片与 PDF 参数

- 图片参数建议传完整 Data URL，例如 `data:image/png;base64,...`
- 本地 PDF 路径支持 `Path("/abs/path/demo.pdf")` 或绝对路径字符串
- 推荐始终使用绝对路径

### 3. 字体与系统依赖

如果使用下面这些场景，建议提前安装模板依赖字体和 Ghostscript：

- 模板打印，尤其是 `image=True`
- 图片打印
- PDF 打印

否则在不同机器上，可能出现字体缺失、排版差异或 PDF 渲染异常。

### 4. 多页 PDF 打印

下面两个接口会按页处理 PDF，并在返回值 `data` 中返回逐页结果：

- `client.tsplPdfsPrint(request)`
- `client.escPdfsPrint(request)`

### 5. 错误码说明

- 下文错误码来自开放平台现有说明文档与当前示例整理
- 某些接口在公开文档中没有单独列出专属错误码，这种情况下请以实际返回的 `code` 和 `message` 为准
- 同一个错误码在不同接口中的含义通常保持一致

<a id="api-列表"></a>
## 三、API 列表

| API | 描述 |
| --- | --- |
| [`BindDeviceRequest`](#1-绑定设备) | 绑定设备，适用机型：所有快麦机型 |
| [`UnbindDeviceRequest`](#2-解绑设备) | 解绑设备，适用机型：所有快麦机型 |
| [`QueryDeviceStatusRequest`](#3-查询设备状态) | 查询设备状态，适用机型：所有快麦机型 |
| [`TsplTemplatePrintRequest`](#4-标签模板打印-间隙纸) | 标签模板打印-间隙纸，适用机型：KM118 系列、KME31 系列、KME41 系列、KME20 系列、KMSX 系列 |
| [`EscTemplatePrintRequest`](#5-小票模板打印-连续纸) | 小票模板打印-连续纸，适用机型：KM118 系列、KME31 系列、KME41 系列、KME20 系列 |
| [`TsplTemplateWriteRequest`](#6-小票模板打印-间隙纸) | 小票模板打印-间隙纸，适用机型：KM118 系列、KME31 系列、KME41 系列、KME20 系列 |
| [`TsplXmlWriteRequest`](#7-自定义-xml-打印-间隙纸) | 自定义 XML 打印-间隙纸 |
| [`EscXmlWriteRequest`](#8-自定义-xml-打印-连续纸) | 自定义 XML 打印-连续纸 |
| [`TsplImageRequest`](#9-图片直接打印-间隙纸) | 图片直接打印-间隙纸 |
| [`EscImageRequest`](#10-图片直接打印-连续纸) | 图片直接打印-连续纸 |
| [`TsplPdfPrintRequest`](#11-pdf-直接打印-间隙纸) | PDF 直接打印-间隙纸 |
| [`EscPdfPrintRequest`](#12-pdf-直接打印-连续纸) | PDF 直接打印-连续纸 |
| [`ResultRequest`](#13-打印任务结果查询) | 打印任务结果查询 |
| [`BroadcastRequest`](#14-语音播报) | 语音播报 |
| [`CancelJobRequest`](#15-取消待打印任务) | 取消待打印任务 |
| [`AdjustDeviceDensityRequest`](#16-调节设备浓度) | 调节设备浓度 |
| [`GetCainiaoCodeRequest`](#17-km360c-获取云打印机-code) | KM360C 获取云打印机 code |
| [`CainiaoBindRequest`](#18-km360c-绑定云打印机) | KM360C 绑定云打印机 |
| [`CainiaoPrintRequest`](#19-km360c-图片打印) | KM360C 图片打印 |

## 四、接口说明

### 1. 绑定设备

**`BindDeviceRequest`**

#### 参数说明

| 参数名 | 含义 | 类型 | 必传 | 说明 |
| --- | --- | --- | --- | --- |
| `sn` | 序列号 | `str` | 是 | 例如 `"KM118DW123"` |
| `deviceKey` | 设备密钥 | `str` | 否 | 若设备机身贴有密钥，通常需要传 |
| `bindName` | 绑定名称 | `str` | 否 | 应用侧自定义名称，例如用户 ID |
| `noteName` | 备注名称 | `str` | 否 | 例如门店、仓库等说明信息 |

#### Python 示例

```python
from kuaimai_core.request import BindDeviceRequest

request = BindDeviceRequest(
    sn="KM118DW123",
    deviceKey="123456",
)
response = client.get_acs_response(request)
```

#### 错误码

| code | 描述 | 处理方式 |
| --- | --- | --- |
| `4001` | 该序列号不存在或未激活 | 检查序列号是否与机器底部或自检页一致；Wi-Fi 机型检查是否已配网；4G 机型检查是否已开机联网 |
| `4009` | 设备密钥不正确 | 检查机器底部设备密钥是否填写正确 |

[↑ 返回API列表](#api-列表)

### 2. 解绑设备

**`UnbindDeviceRequest`**

#### 参数说明

| 参数名 | 含义 | 类型 | 必传 | 说明 |
| --- | --- | --- | --- | --- |
| `sn` | 序列号 | `str` | 是 | 例如 `"KM118DW123"` |
| `deviceKey` | 设备密钥 | `str` | 否 | 若设备机身贴有密钥，通常需要传 |

#### Python 示例

```python
from kuaimai_core.request import UnbindDeviceRequest

request = UnbindDeviceRequest(
    sn="KM118DW123",
    deviceKey="123456",
)
response = client.get_acs_response(request)
```

#### 错误码

| code | 描述 | 处理方式 |
| --- | --- | --- |
| `4001` | 该序列号不存在或未激活 | 检查序列号是否与机器底部或自检页一致；Wi-Fi 机型检查是否已配网；4G 机型检查是否已开机联网 |
| `4009` | 设备密钥不正确 | 检查机器底部设备密钥是否填写正确 |

[↑ 返回API列表](#api-列表)

### 3. 查询设备状态

**`QueryDeviceStatusRequest`**

一次请求最多查询 100 个设备。

#### 参数说明

| 参数名 | 含义 | 类型 | 必传 | 说明 |
| --- | --- | --- | --- | --- |
| `sns` | 序列号数组字符串 | `str` | 是 | 例如 `json.dumps(["KM118DW123"])` |

#### Python 示例

```python
import json

from kuaimai_core.request import QueryDeviceStatusRequest

request = QueryDeviceStatusRequest(
    sns=json.dumps(["KM118DW123"], ensure_ascii=False),
)
response = client.get_acs_response(request)
```

#### 常见返回字段

| 字段 | 含义 |
| --- | --- |
| `data[].sn` | 序列号 |
| `data[].status` | `ONLINE` 在线，`OFFLINE` 离线，`UNACTIVE` 未激活，`DISABLE` 已禁用 |

#### 错误码

公开文档未单独列出该接口专属错误码；调用失败时请以接口实际返回的 `code` 和 `message` 为准。

[↑ 返回API列表](#api-列表)

### 4. 标签模板打印-间隙纸

**`TsplTemplatePrintRequest`**

适用机型：KM118 系列、KME31 系列、KME41 系列、KME20 系列、KMSX 系列。

#### 参数说明

| 参数名 | 含义 | 类型 | 必传 | 说明 |
| --- | --- | --- | --- | --- |
| `sn` | 序列号 | `str` | 条件必传 | 普通云打印场景传 `sn` |
| `templateId` | 模板 ID | `int` | 是 | 模板管理中的模板 ID |
| `renderDataArray` | 动态渲染数据 | `str` | 常用 | JSON 数组字符串，支持一次渲染多份数据 |
| `renderDataJsonArray` | 动态渲染数据 | `list` / `Any` | 否 | 也可直接传 Python 对象 |
| `printTimes` | 打印份数 | `int` | 否 | 默认 1 |
| `image` | 是否图片方式渲染 | `bool` | 否 | `True` 时运行机器需安装模板对应字体 |
| `dpi` | 分辨率 | `int` | 否 | 常见值 `203` / `300` |
| `imei` | KM360C 专用 | `str` | 条件必传 | KM360C 场景可只传 `imei` 不传 `sn` |

#### Python 示例

```python
from kuaimai_core.request import TsplTemplatePrintRequest

request = TsplTemplatePrintRequest(
    sn="KM118DW123",
    templateId=1634989639,
    renderDataArray='[{"table_test":[{"key_test":"3449394"}]}]',
    printTimes=1,
    image=True,
)
response = client.get_acs_response(request)
```

#### KM360C 场景示例

```python
from kuaimai_core.request import TsplTemplatePrintRequest

request = TsplTemplatePrintRequest(
    imei="863130068516971",
    templateId=1634989639,
    renderDataArray='[{"table_test":[{"key_test":"3449394"}]}]',
)
response = client.get_acs_response(request)
```

#### 错误码

| code | 描述 | 处理方式 |
| --- | --- | --- |
| `4001` | 该序列号不存在或未激活 | 检查序列号、配网状态和设备开机状态 |
| `4005` | 设备离线 | 检查设备联网状态 |
| `6009` | 动态渲染数据结构错误 | 检查 `renderDataArray` 的 JSON 结构是否符合模板要求 |
| `6022` | 动态批量渲染数量过大 | 控制单次动态批量渲染数量小于等于 50 |
| `6015` | 无权限模板访问 | 检查 `templateId` 是否属于当前 `appid` 创建的模板 |
| `6010` | 模板类型不匹配 | 检查 `templateId` 是否与当前打印接口类型一致 |

[↑ 返回API列表](#api-列表)

### 5. 小票模板打印-连续纸

**`EscTemplatePrintRequest`**

#### 参数说明

| 参数名 | 含义 | 类型 | 必传 | 说明 |
| --- | --- | --- | --- | --- |
| `sn` | 序列号 | `str` | 是 | 打印机序列号 |
| `templateId` | 模板 ID | `int` | 是 | 模板管理中的模板 ID |
| `renderData` | 动态渲染数据 | `str` | 常用 | 直接传 JSON 字符串 |
| `renderDataJson` | 动态渲染数据 | `dict` / `Any` | 否 | 也可直接传 Python 对象 |
| `cut` | 是否切纸 | `bool` | 否 | 是否切纸依赖设备能力 |
| `copies` | 打印份数 | `int` | 否 | 按需传入 |
| `endFeed` | 末尾走纸 | `int` | 否 | 连续纸场景常用 |

#### Python 示例

```python
from kuaimai_core.request import EscTemplatePrintRequest

request = EscTemplatePrintRequest(
    sn="KM118DW123",
    templateId=1634960019,
    renderData='{"table_test":[{"key_test":"3449394"}]}',
)
response = client.get_acs_response(request)
```

#### 错误码

| code | 描述 | 处理方式 |
| --- | --- | --- |
| `4001` | 该序列号不存在或未激活 | 检查序列号、配网状态和设备开机状态 |
| `4005` | 设备离线 | 检查设备联网状态 |
| `6009` | 动态渲染数据结构错误 | 检查 `renderData` 的 JSON 结构是否正确 |
| `6015` | 无权限模板访问 | 检查 `templateId` 是否属于当前 `appid` 创建的模板 |
| `6010` | 模板类型不匹配 | 检查 `templateId` 是否与当前打印接口类型一致 |

[↑ 返回API列表](#api-列表)

### 6. 小票模板打印-间隙纸

**`TsplTemplateWriteRequest`**

#### 参数说明

| 参数名 | 含义 | 类型 | 必传 | 说明 |
| --- | --- | --- | --- | --- |
| `sn` | 序列号 | `str` | 是 | 打印机序列号 |
| `templateId` | 模板 ID | `int` | 是 | 模板 ID |
| `renderData` | 动态渲染数据 | `str` | 常用 | 直接传 JSON 字符串 |
| `renderDataJson` | 动态渲染数据 | `dict` / `Any` | 否 | 也可直接传 Python 对象 |
| `printTimes` | 打印份数 | `int` | 否 | 默认 1 |

#### Python 示例

```python
from kuaimai_core.request import TsplTemplateWriteRequest

request = TsplTemplateWriteRequest(
    sn="KM118DW123",
    templateId=1634960019,
    renderData='{"table_test":[{"key_test":"3449394"}]}',
    printTimes=1,
)
response = client.get_acs_response(request)
```

#### 错误码

| code | 描述 | 处理方式 |
| --- | --- | --- |
| `4001` | 该序列号不存在或未激活 | 检查序列号、配网状态和设备开机状态 |
| `4005` | 设备离线 | 检查设备联网状态 |
| `6009` | 动态渲染数据结构错误 | 检查 `renderData` 的 JSON 结构是否正确 |
| `6015` | 无权限模板访问 | 检查 `templateId` 是否属于当前 `appid` 创建的模板 |
| `6010` | 模板类型不匹配 | 检查 `templateId` 是否与当前打印接口类型一致 |

[↑ 返回API列表](#api-列表)

### 7. 自定义 XML 打印-间隙纸

**`TsplXmlWriteRequest`**

#### 参数说明

| 参数名 | 含义 | 类型 | 必传 | 说明 |
| --- | --- | --- | --- | --- |
| `sn` | 序列号 | `str` | 是 | 打印机序列号 |
| `xmlStr` | XML 内容 | `str` | `xmlStr` / `jobs` 二选一 | 自定义 XML 指令内容 |
| `jobs` | 批量 XML 数组 | `list` / `Any` | `xmlStr` / `jobs` 二选一 | 一次最多 20 单，KMSX320 最多 5 单 |
| `image` | 是否包含图片 | `bool` | 否 | 默认按普通内容处理 |
| `printTimes` | 打印份数 | `int` | 否 | 默认 1 |

#### Python 示例

```python
from kuaimai_core.request import TsplXmlWriteRequest

request = TsplXmlWriteRequest(
    sn="KM118DW123",
    xmlStr='<page><render width="100" height="150"><t x="20" y="20">生产加工单</t></render></page>',
    printTimes=1,
)
response = client.get_acs_response(request)
```

#### 错误码

| code | 描述 | 处理方式 |
| --- | --- | --- |
| `4001` | 该序列号不存在或未激活 | 检查序列号、配网状态和设备开机状态 |
| `4005` | 设备离线 | 检查设备联网状态 |
| `6024` | XML 数据过大 | 如果使用批量打印，控制数组长度小于等于 20 |

[↑ 返回API列表](#api-列表)

### 8. 自定义 XML 打印-连续纸

**`EscXmlWriteRequest`**

#### 参数说明

| 参数名 | 含义 | 类型 | 必传 | 说明 |
| --- | --- | --- | --- | --- |
| `sn` | 序列号 | `str` | 是 | 打印机序列号 |
| `instructions` | XML / 指令内容 | `str` | 是 | 连续纸打印指令 |
| `volume` | 音量 | `int` | 否 | 仅支持带语音的设备 |
| `volumeIndex` | 语音索引 | `int` | 否 | 仅支持带语音的设备 |
| `cut` | 是否切刀 | `bool` / `int` | 否 | 视设备能力而定 |

#### Python 示例

```python
from kuaimai_core.request import EscXmlWriteRequest

request = EscXmlWriteRequest(
    sn="KM118DW123",
    instructions="<page><render><t size='01' feed='9'>hello,word</t></render></page>",
)
response = client.get_acs_response(request)
```

#### 错误码

| code | 描述 | 处理方式 |
| --- | --- | --- |
| `4001` | 该序列号不存在或未激活 | 检查序列号、配网状态和设备开机状态 |
| `4005` | 设备离线 | 检查设备联网状态 |

[↑ 返回API列表](#api-列表)

### 9. 图片直接打印-间隙纸

**`TsplImageRequest`**

#### 参数说明

| 参数名 | 含义 | 类型 | 必传 | 说明 |
| --- | --- | --- | --- | --- |
| `sn` | 序列号 | `str` | 是 | 打印机序列号 |
| `imageBase64` | 图片 base64 | `str` | 常用 | 推荐传完整 Data URL |
| `bufferedImage` | 内存图片对象 | `Any` | 否 | SDK 内部转换场景可用 |
| `printTimes` | 打印份数 | `int` | 否 | 默认 1 |
| `setWidth` | 标签宽度 | `int` | 否 | 单位 mm |
| `setHeight` | 标签高度 | `int` | 否 | 单位 mm |
| `dpi` | 分辨率 | `int` | 否 | 常见值 `203` / `300` |

#### Python 示例

```python
from kuaimai_core.request import TsplImageRequest

request = TsplImageRequest(
    sn="KM118DW123",
    imageBase64="data:image/png;base64,......",
    printTimes=1,
)
response = client.get_acs_response(request)
```

#### 错误码

| code | 描述 | 处理方式 |
| --- | --- | --- |
| `4001` | 该序列号不存在或未激活 | 检查序列号、配网状态和设备开机状态 |
| `4005` | 设备离线 | 检查设备联网状态 |
| `6019` | 打印内容过大 | 减少图片数据大小，建议控制在平台限制内 |

[↑ 返回API列表](#api-列表)

### 10. 图片直接打印-连续纸

**`EscImageRequest`**

#### 参数说明

| 参数名 | 含义 | 类型 | 必传 | 说明 |
| --- | --- | --- | --- | --- |
| `sn` | 序列号 | `str` | 是 | 打印机序列号 |
| `imageBase64` | 图片 base64 | `str` | 常用 | 推荐传完整 Data URL |
| `bufferedImage` | 内存图片对象 | `Any` | 否 | SDK 内部转换场景可用 |
| `printWidth` | 打印宽度 | `int` | 否 | 常用值 `58` / `80` |
| `endFeed` | 末尾走纸 | `int` | 否 | 默认 3 |

#### Python 示例

```python
from kuaimai_core.request import EscImageRequest

request = EscImageRequest(
    sn="KM118DW123",
    imageBase64="data:image/png;base64,......",
    printWidth=58,
)
response = client.get_acs_response(request)
```

#### 错误码

| code | 描述 | 处理方式 |
| --- | --- | --- |
| `4001` | 该序列号不存在或未激活 | 检查序列号、配网状态和设备开机状态 |
| `4005` | 设备离线 | 检查设备联网状态 |
| `6019` | 打印内容过大 | 减少图片数据大小，建议控制在平台限制内 |

[↑ 返回API列表](#api-列表)

### 11. PDF 直接打印-间隙纸

**`TsplPdfPrintRequest`**

#### 参数说明

| 参数名 | 含义 | 类型 | 必传 | 说明 |
| --- | --- | --- | --- | --- |
| `sn` | 序列号 | `str` | 是 | 打印机序列号 |
| `file` | PDF 文件 | `str` / `Path` | 是 | 本地 PDF 路径 |
| `width` | 标签宽度 | `int` | 否 | 单位 mm，默认 75 |
| `height` | 标签高度 | `int` | 否 | 单位 mm，默认 100 |
| `dpi` | 分辨率 | `int` | 否 | 默认 203 |

#### Python 示例-单页

```python
from pathlib import Path

from kuaimai_core.request import TsplPdfPrintRequest

request = TsplPdfPrintRequest(
    sn="KM118DW123",
    file=Path("/pdf/demo.pdf"),
)
response = client.get_acs_response(request)
```

#### Python 示例-多页

```python
from pathlib import Path

from kuaimai_core.request import TsplPdfPrintRequest

request = TsplPdfPrintRequest(
    sn="KM118DW123",
    file=Path("/pdf/demo.pdf"),
)
response = client.tsplPdfsPrint(request)
```

#### 错误码

| code | 描述 | 处理方式 |
| --- | --- | --- |
| `4001` | 该序列号不存在或未激活 | 检查序列号、配网状态和设备开机状态 |
| `4005` | 设备离线 | 检查设备联网状态 |
| `6019` | 打印内容过大 | 减少 PDF 渲染后数据大小，必要时缩小尺寸或拆页打印 |

[↑ 返回API列表](#api-列表)

### 12. PDF 直接打印-连续纸

**`EscPdfPrintRequest`**

#### 参数说明

| 参数名 | 含义 | 类型 | 必传 | 说明 |
| --- | --- | --- | --- | --- |
| `sn` | 序列号 | `str` | 是 | 打印机序列号 |
| `file` | PDF 文件 | `str` / `Path` | 是 | 本地 PDF 路径 |
| `printWidth` | 打印宽度 | `int` | 否 | 常用值 `58` / `80` |
| `endFeed` | 末尾走纸 | `int` | 否 | 默认 3 |

#### Python 示例-单页

```python
from pathlib import Path

from kuaimai_core.request import EscPdfPrintRequest

request = EscPdfPrintRequest(
    sn="KM118DW123",
    file=Path("/pdf/demo.pdf"),
    printWidth=58,
)
response = client.get_acs_response(request)
```

#### Python 示例-多页

```python
from pathlib import Path

from kuaimai_core.request import EscPdfPrintRequest

request = EscPdfPrintRequest(
    sn="KM118DW123",
    file=Path("/pdf/demo.pdf"),
    printWidth=58,
)
response = client.escPdfsPrint(request)
```

#### 错误码

| code | 描述 | 处理方式 |
| --- | --- | --- |
| `4001` | 该序列号不存在或未激活 | 检查序列号、配网状态和设备开机状态 |
| `4005` | 设备离线 | 检查设备联网状态 |
| `6019` | 打印内容过大 | 减少 PDF 渲染后数据大小，必要时缩小尺寸或拆页打印 |

[↑ 返回API列表](#api-列表)

### 13. 打印任务结果查询

**`ResultRequest`**

建议同时在平台后台配置打印结果回调地址，打印完成后平台可主动推送结果。

#### 参数说明

| 参数名 | 含义 | 类型 | 必传 | 说明 |
| --- | --- | --- | --- | --- |
| `sn` | 序列号 | `str` | 是 | 打印机序列号 |
| `jobIds` | 任务 ID 列表 | `list` / `Any` | 是 | 推荐直接传 Python 列表 |
| `jobIdStr` | 任务 ID 字符串 | `str` | 否 | 按需使用 |

同一个序列号一秒钟最多查询两次，一次最多查询 50 个任务 ID。

#### Python 示例

```python
from kuaimai_core.request import ResultRequest

request = ResultRequest(
    sn="KM118DW123",
    jobIds=["1775636782780"],
)
response = client.get_acs_response(request)
```

#### 常见返回字段

| 字段 | 含义 |
| --- | --- |
| `data[].jobId` | 打印任务 ID |
| `data[].desc` | 结果描述 |
| `data[].code` | `2000` 打印成功，`2006` 打印等待中，`2007` 打印失败，`2004` 打印异常 |

#### 错误码

| code | 描述 | 处理方式 |
| --- | --- | --- |
| `6026` | 打印结果查询超过限制 | 控制单次查询的任务 ID 数量小于等于 50 |
| `6021` | 该序列号查询过于频繁 | 降低查询频率，避免对同一序列号高频查询 |

[↑ 返回API列表](#api-列表)

### 14. 语音播报

**`BroadcastRequest`**

适用机型：KM118MGL、KME31GP。

#### 参数说明

| 参数名 | 含义 | 类型 | 必传 | 说明 |
| --- | --- | --- | --- | --- |
| `sn` | 序列号 | `str` | 是 | 打印机序列号 |
| `volume` | 音量 | `int` | 是 | 范围 `0~99` |
| `volumeContent` | 播报内容 | `str` | 是 | 最大 50 个字符 |

#### Python 示例

```python
from kuaimai_core.request import BroadcastRequest

request = BroadcastRequest(
    sn="KM118DW123",
    volume=80,
    volumeContent="测试语言播报",
)
response = client.get_acs_response(request)
```

#### 错误码

| code | 描述 | 处理方式 |
| --- | --- | --- |
| `6036` | 语言播报内容超过限制 | 控制播报内容长度小于等于 50 个字符 |
| `6035` | 该机型不支持语言播报 | 使用支持语音播报的机型 |
| `4005` | 设备离线 | 检查设备联网状态 |

[↑ 返回API列表](#api-列表)

### 15. 取消待打印任务

**`CancelJobRequest`**

#### 参数说明

| 参数名 | 含义 | 类型 | 必传 | 说明 |
| --- | --- | --- | --- | --- |
| `sn` | 序列号 | `str` | 是 | 打印机序列号 |

#### Python 示例

```python
from kuaimai_core.request import CancelJobRequest

request = CancelJobRequest(sn="KM118DW123")
response = client.get_acs_response(request)
```

#### 错误码

公开文档未单独列出该接口专属错误码；调用失败时请以接口实际返回的 `code` 和 `message` 为准。

[↑ 返回API列表](#api-列表)

### 16. 调节设备浓度

**`AdjustDeviceDensityRequest`**

适用机型：KM118 系列、KME31 系列、KME41 系列，且设备需要处于标签模式（自检页指令为 `tspl`）。

#### 参数说明

| 参数名 | 含义 | 类型 | 必传 | 说明 |
| --- | --- | --- | --- | --- |
| `sn` | 序列号 | `str` | 是 | 打印机序列号 |
| `density` | 浓度值 | `int` | 是 | 范围 `1~15`，推荐默认值 `8` |

#### Python 示例

```python
from kuaimai_core.request import AdjustDeviceDensityRequest

request = AdjustDeviceDensityRequest(
    sn="KM118DW123",
    density=8,
)
response = client.get_acs_response(request)
```

#### 错误码

公开文档未单独列出该接口专属错误码；调用失败时请以接口实际返回的 `code` 和 `message` 为准。

[↑ 返回API列表](#api-列表)

### 17. KM360C 获取云打印机 code

**`GetCainiaoCodeRequest`**

#### 参数说明

| 参数名 | 含义 | 类型 | 必传 | 说明 |
| --- | --- | --- | --- | --- |
| `imei` | 设备 IMEI | `str` | 是 | 可在设备自检页查看 |

#### Python 示例

```python
from kuaimai_core.request import GetCainiaoCodeRequest

request = GetCainiaoCodeRequest(
    imei="863130068516971",
)
response = client.get_acs_response(request)
```

说明：返回的 code 有效期通常为 5 分钟。

#### 错误码

公开文档未单独列出该接口专属错误码；调用失败时请以接口实际返回的 `code` 和 `message` 为准。

[↑ 返回API列表](#api-列表)

### 18. KM360C 绑定云打印机

**`CainiaoBindRequest`**

#### 参数说明

| 参数名 | 含义 | 类型 | 必传 | 说明 |
| --- | --- | --- | --- | --- |
| `imei` | 设备 IMEI | `str` | 是 | 可在设备自检页查看 |
| `code` | 绑定 code | `str` | 是 | 通过获取 code 接口拿到 |

#### Python 示例

```python
from kuaimai_core.request import CainiaoBindRequest

request = CainiaoBindRequest(
    imei="863130068516971",
    code="4841",
)
response = client.get_acs_response(request)
```

#### 错误码

公开文档未单独列出该接口专属错误码；调用失败时请以接口实际返回的 `code` 和 `message` 为准。

[↑ 返回API列表](#api-列表)

### 19. KM360C 图片打印

**`CainiaoPrintRequest`**

#### 参数说明

| 参数名 | 含义 | 类型 | 必传 | 说明 |
| --- | --- | --- | --- | --- |
| `imei` | 设备 IMEI | `str` | 是 | 可在设备自检页查看 |
| `imageBase64Data` | 图片 base64 | `str` | 是 | 建议传完整 Data URL |

#### Python 示例

```python
from kuaimai_core.request import CainiaoPrintRequest

request = CainiaoPrintRequest(
    imei="863130068516971",
    imageBase64Data="data:image/png;base64,......",
)
response = client.get_acs_response(request)
```

#### 错误码

公开文档未单独列出该接口专属错误码；调用失败时请以接口实际返回的 `code` 和 `message` 为准。

[↑ 返回API列表](#api-列表)

## 五、与 CloudExample.py 的对应关系

当前 demo 中的示例函数和请求类型对应关系如下：

| `CloudExample.py` 中的函数 | 对应请求类型 / 调用方式 |
| --- | --- |
| `bind_device_example` | `BindDeviceRequest` |
| `unbind_device_example` | `UnbindDeviceRequest` |
| `query_device_status_example` | `QueryDeviceStatusRequest` |
| `esc_template_print_example` | `EscTemplatePrintRequest` |
| `tspl_template_print_example` | `TsplTemplatePrintRequest` |
| `tspl_template_write_example` | `TsplTemplateWriteRequest` |
| `tspl_xml_write_example` | `TsplXmlWriteRequest` |
| `esc_xml_write_example` | `EscXmlWriteRequest` |
| `result_example` | `ResultRequest` |
| `tspl_image_print_example` | `TsplImageRequest` |
| `tspl_pdf_print_example` | `TsplPdfPrintRequest` + `get_acs_response` |
| `tspl_pdfs_print_example` | `TsplPdfPrintRequest` + `client.tsplPdfsPrint(...)` |
| `esc_image_print_example` | `EscImageRequest` |
| `esc_pdf_print_example` | `EscPdfPrintRequest` + `get_acs_response` |
| `esc_pdfs_print_example` | `EscPdfPrintRequest` + `client.escPdfsPrint(...)` |
| `broadcast_example` | `BroadcastRequest` |
| `cancel_job_example` | `CancelJobRequest` |
| `adjust_device_density_example` | `AdjustDeviceDensityRequest` |
| `get_cainiao_code_example` | `GetCainiaoCodeRequest` |
| `cainiao_bind_example` | `CainiaoBindRequest` |
| `cainiao_print_example` | `CainiaoPrintRequest` |
| `cainiao_tspl_template_print_example` | `TsplTemplatePrintRequest(imei=...)` |

## 六、补充建议

- 模板打印时，建议先在平台确认模板字段名、表名和当前请求中的渲染数据完全一致
- 图片和 PDF 打印若遇到“内容过大”，优先检查原图尺寸、页尺寸和渲染后的 base64 大小
- 对同一设备进行结果查询时，注意控制频率，避免命中限流
- 真实对接时建议先从 `CloudExample.py` 中打开一个最简单的函数验证，再逐步切换到业务场景
