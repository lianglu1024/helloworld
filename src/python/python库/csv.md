# CSV

## 读文件
```python
with open('1067order.csv', 'r+', encoding='utf-8') as f:
    reader = csv.reader(f)
    for item in reader:  
        order_name = item[0]  # 第一列
        status_index = item[1]  # 第二列
```
## 写文件
```python
with open("order.csv", "w+") as f:
    writer = csv.writer(f)
    row1 = ["order_name", "shipping_country"]
    row2 = ["SO123456", "India"]
    row3 = ["SO123457", "American"]
    writer.writerow(row1)
    writer.writerow(row2)
    writer.writerow(row3)
```
