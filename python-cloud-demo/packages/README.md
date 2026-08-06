# packages

内部交付时，这个目录必须同时包含两个 SDK wheel 和 Python 3.6 使用的
`qrcode` 预构建 wheel：

```text
python_kuaimai_core-0.1.1-py3-none-any.whl
qrcode-7.3.1-py3-none-any.whl
python_kuaimai_core_legacy-0.1.1-py3-none-any.whl
```

不要把 wheel 改名成 `latest` 之类的别名，`pip` 会把它识别成不合法的 wheel 文件名。

复制完成后，同步更新两个 requirements 文件：

```bash
pip install -r requirements.txt         # Python 3.10+
pip install -r requirements-py36.txt    # Python 3.6–3.9
```

`requirements-py36.txt` 必须先列出本地 `qrcode` wheel，再列出 Legacy
SDK wheel。`qrcode 7.3.1` 在 PyPI 上没有官方 wheel，如果缺少这个文件，
客户的 Python 3.6 环境会尝试运行源码包的 `setup.py`。
