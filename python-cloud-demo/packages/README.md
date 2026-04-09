# packages

内部交付时，把 `python-kuaimai-core` 构建出来的 wheel 原样复制到这个目录，例如：

`python_kuaimai_core-0.1.0-py3-none-any.whl`

不要把 wheel 改名成 `latest` 之类的别名，`pip` 会把它识别成不合法的 wheel 文件名。

复制完成后，同步更新 `python-cloud-demo/requirements.txt`，让客户直接执行：

```bash
pip install -r requirements.txt
```
