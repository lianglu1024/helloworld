# Httpclient

1.依赖包

```xml
<dependency>
  <groupId>org.apache.httpcomponents</groupId>
  <artifactId>httpclient</artifactId>
  <version>4.5.9</version>
</dependency>
```

2.GET请求，带查询参数

```java
package com.zstu.springbootstudy;

import org.apache.http.HttpEntity;
import org.apache.http.NameValuePair;
import org.apache.http.client.ClientProtocolException;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.utils.URIBuilder;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.util.EntityUtils;
import org.junit.jupiter.api.Test;

import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.util.*;

public class HttpTest {

    @Test
    public void http() throws Exception{
        CloseableHttpResponse response = null;
        CloseableHttpClient httpclient = HttpClients.createDefault();

        URIBuilder uriBuilder=new URIBuilder("http://127.0.0.1:4002/sale/server/seller/settlement");
        //设置参数
        uriBuilder.setParameter("order_name","SO12345");
        System.out.println(uriBuilder.build());
        HttpGet httpGet = new HttpGet(uriBuilder.build());

        String result = "";

        try {
            RequestConfig requestConfig = RequestConfig.custom().setConnectTimeout(35000)// 连接主机服务超时时间
                    .setConnectionRequestTimeout(35000)// 请求超时时间
                    .setSocketTimeout(60000)// 数据读取超时时间
                    .build();
            httpGet.setConfig(requestConfig);
            httpGet.addHeader("Content-Type", "application/json");
            httpGet.addHeader("Authorization", "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9k82YW-o9qe6o43K2Uvq8bZhDzRLNyMGAw09ZTILU76E");


            response = httpclient.execute(httpGet);
            System.out.println(response.getStatusLine().getStatusCode());  // 200
            HttpEntity entity = response.getEntity();
            result = EntityUtils.toString(entity);
        } finally {
            response.close();
        }

        System.out.println(result); // {"success": false, "message": "Seller仓没有找到订单SO12345的信息"}
    }
}
```

3.POST请求，请求体是json

```java
package com.zstu.springbootstudy;

import org.apache.http.HttpEntity;
import org.apache.http.NameValuePair;
import org.apache.http.client.ClientProtocolException;
import org.apache.http.client.config.RequestConfig;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.util.EntityUtils;
import org.junit.jupiter.api.Test;

import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.util.*;

public class HttpTest {

    @Test
    public void http() throws Exception{
        CloseableHttpResponse response = null;
        CloseableHttpClient httpclient = HttpClients.createDefault();
        HttpPost httpPost = new HttpPost("http://127.0.0.1:4002/wms/logistics/pickup_address");
        String result = "";

        try {
            RequestConfig requestConfig = RequestConfig.custom().setConnectTimeout(35000)// 连接主机服务超时时间
                    .setConnectionRequestTimeout(35000)// 请求超时时间
                    .setSocketTimeout(60000)// 数据读取超时时间
                    .build();
            httpPost.setConfig(requestConfig);
            httpPost.addHeader("Content-Type", "application/json");
            httpPost.addHeader("Authorization", "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9k82YW-o9qe6o43K2Uvq8bZhDzRLNyMGAw09ZTILU76E");

            StringEntity requestEntity = new StringEntity("{\"fo\":\"SO123\"}","utf-8");
            httpPost.setEntity(requestEntity);

            response = httpclient.execute(httpPost);
            System.out.println(response.getStatusLine().getStatusCode());  // 200
            HttpEntity entity = response.getEntity();
            result = EntityUtils.toString(entity);
        } finally {
            response.close();
        }

        System.out.println(result); // {"success": false, "message": "india_seller仓没有找到SO123信息"}
    }
}
```

