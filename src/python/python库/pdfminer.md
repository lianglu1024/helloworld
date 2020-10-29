# pdfminer
pdfminer包是用来读取pdf文件内容的包

## 安装：
```
pip3 install pdfminer.six
```

## 基本使用：
```python
from urllib.request import urlopen
from io import BytesIO
from pdfminer.high_level import extract_text
# 读取网络文件pdf
pdf_url = "http://xxx.pdf"
res = urlopen(pdf_url)
f = BytesIO(res.read())

# 读取本地文件
f = open("xxx.pdf", "rb")
pdf_content = extract_text(f)

f.close()
```
