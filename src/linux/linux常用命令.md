# Linux常用命令

* `grep abc abc.log [-C 10]`：查找关键字abc在abc.log中所在行信息。`-C 10`表示查出行的上下10行信息
* `zgrep abc abc.gz`：查找关键字abc在压缩文件abc.gz中所在行信息

* curl

post请求：curl -H "Authorization:12345" -H "Content-Type:application/json" -d '{"carrier_list":["Ecom","Delhivery"]}' http://india-seller.erp.test.yuceyi.com:5678/sale/marketplace_order/address_register

