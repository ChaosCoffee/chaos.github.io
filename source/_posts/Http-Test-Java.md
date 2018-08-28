---
title: Java Http工具类
comments: false
date: 2018-08-23 15:52:44
categories: programmes
tags: java
img:
---

# Java Http工具类  

<!-- TOC -->

- [Java Http工具类](#java-http%E5%B7%A5%E5%85%B7%E7%B1%BB)
    - [引入包概览](#%E5%BC%95%E5%85%A5%E5%8C%85%E6%A6%82%E8%A7%88)
    - [Apache Http util](#apache-http-util)
        - [maven依赖](#maven%E4%BE%9D%E8%B5%96)
        - [代码实例](#%E4%BB%A3%E7%A0%81%E5%AE%9E%E4%BE%8B)
    - [OkHttp Http util](#okhttp-http-util)
        - [OkHttp 介绍](#okhttp-%E4%BB%8B%E7%BB%8D)
        - [maven依赖](#maven%E4%BE%9D%E8%B5%96)
        - [代码实例](#%E4%BB%A3%E7%A0%81%E5%AE%9E%E4%BE%8B)

<!-- /TOC -->

## 引入包概览
``` java
import com.alibaba.fastjson.JSONObject;
import com.google.common.collect.Maps;
import okhttp3.*;
import org.apache.http.HttpEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.client.utils.URIBuilder;
import org.apache.http.entity.ContentType;
import org.apache.http.entity.StringEntity;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.impl.conn.PoolingHttpClientConnectionManager;
import org.apache.http.util.EntityUtils;

import java.io.IOException;
import java.net.URISyntaxException;
import java.util.Map;

```

## Apache Http util

### maven依赖

``` xml
-- maven dependency

<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>fastjson</artifactId>
  <version>1.2.49</version>
</dependency>
     
<dependency>
  <groupId>org.apache.httpcomponents</groupId>
  <artifactId>httpclient</artifactId>
  <version>4.5.6</version>
</dependency>

<dependency>
  <groupId>org.apache.httpcomponents</groupId>
  <artifactId>httpcore</artifactId>
  <version>4.4.10</version>
</dependency>

```
### 代码实例
     
``` java
    private static final String UTF_8 = "UTF-8";

    private static CloseableHttpClient client;

    static {
        PoolingHttpClientConnectionManager cm = new PoolingHttpClientConnectionManager();
        cm.setMaxTotal(100);
        cm.setDefaultMaxPerRoute(20);
        cm.setDefaultMaxPerRoute(50);
        client = HttpClients.custom().setConnectionManager(cm).build();
    }

    /**
     * Apache Http Get
     *
     * @param url
     * @param params
     * @param headers
     * @return
     */
    public Object doGet(String url, Map<String, String> params, Map<String, String> headers) {
        URIBuilder builder;
        try {
            builder = new URIBuilder(url);
        } catch (URISyntaxException e) {
            e.printStackTrace();
            return null;
        }
        params.forEach((k, v) -> builder.addParameter(k, v));
        CloseableHttpResponse response = null;
        try {
            HttpGet httpGet = new HttpGet(builder.toString());
            headers.forEach((k, v) -> httpGet.addHeader(k, v));
            client = HttpClients.createDefault();
            response = client.execute(httpGet);
            HttpEntity httpEntity = response.getEntity();
            String data = null;
            if (null != httpEntity) {
                data = EntityUtils.toString(httpEntity, UTF_8);
                System.out.println("data = " + data);
            }
            return data;
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (response != null) {
                try {
                    response.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return null;
    }

    /**
     * Apache Http Post
     *
     * @param url
     * @param params
     * @param headers
     * @return
     */
    public String doPost(String url, Map<String, String> params, Map<String, String> headers) {
        URIBuilder builder = null;
        try {
            builder = new URIBuilder(url);
        } catch (URISyntaxException e) {
            e.printStackTrace();
        }
        CloseableHttpResponse response = null;
        try {
            HttpPost httpPost = new HttpPost(builder.toString());
            headers.forEach((k, v) -> httpPost.addHeader(k, v));
            JSONObject param = new JSONObject();
            param.forEach((k, v) -> param.put(k, v));
            StringEntity stringEntity = new StringEntity(param.toJSONString(), ContentType.APPLICATION_JSON);
            httpPost.setEntity(stringEntity);
            client = HttpClients.createDefault();
            response = client.execute(httpPost);
            HttpEntity httpEntity = response.getEntity();
            String data = null;
            if (null != httpEntity) {
                data = EntityUtils.toString(httpEntity, UTF_8);
                System.out.println("data = " + data);
            }
            return data;
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (response != null) {
                try {
                    response.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return null;
    }
```

---  
## OkHttp Http util  
### OkHttp 介绍
> OkHttp是一款优秀的HTTP框架，它支持get请求和post请求，支持基于Http的文件上传和下载，支持加载图片，支持下载文件透明的GZIP压缩，支持响应缓存避免重复的网络请求，支持使用连接池来降低响应延迟问题  
> 
### maven依赖

``` xml
<dependency>
  <groupId>com.squareup.okhttp3</groupId>
  <artifactId>okhttp</artifactId>
  <version>3.9.1</version>
</dependency>

```

### 代码实例
``` java
    /**
     * okhttp3 Get
     *
     * @param url
     * @param params
     * @param headers
     */

    public void doOKGet(String url, Map<String, String> params, Map<String, String> headers) {
        try {
            OkHttpClient client = new OkHttpClient();
            HttpUrl.Builder builder = HttpUrl.parse(url)
                    .newBuilder();
            params.forEach((k, v) -> builder.addQueryParameter(k, v));
            Request.Builder request = new Request.Builder()
                    .url(builder.build())
                    .get();
            headers.forEach((k, v) -> request.addHeader(k, v));
            Response response = client.newCall(request.build()).execute();
            System.out.println("response = " + response.isSuccessful());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * okhttp3 Post
     *
     * @param url
     * @param json
     * @throws IOException
     */
    public void doOKPost(String url, String json, Map<String, String> headers) {
        OkHttpClient client = new OkHttpClient();
        RequestBody body = RequestBody.create(MediaType.parse("application/json; charset=utf-8"), json);//json data
        Request.Builder request = new Request.Builder()
                .url(url)
                .post(body)
                .addHeader("Content-Type", "application/json; charset=utf-8");
        headers.forEach((k, v) -> request.addHeader(k, v));//user header data
        try {
            Response response = client.newCall(request.build()).execute();
            System.out.println("response = " + response.isSuccessful());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * okhttp3 Post
     *
     * @param url
     * @param params
     * @param headers
     */
    public void doOKPost(String url, Map<String, String> params, Map<String, String> headers) {
        OkHttpClient client = new OkHttpClient();
        FormBody.Builder body = new FormBody.Builder();//form data
        params.forEach((k, v) -> body.add(k, v));
        Request.Builder request = new Request.Builder()
                .url(url)
                .post(body.build());
        headers.forEach((k, v) -> request.addHeader(k, v));//header data
        try {
            Response response = client.newCall(request.build()).execute();
            System.out.println("response = " + response.isSuccessful());
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        HttpUtils httpUtils = new HttpUtils();
        httpUtils.doOKGet("http://www.baidu.com", Maps.newHashMap(), Maps.newHashMap());
    }
}
```