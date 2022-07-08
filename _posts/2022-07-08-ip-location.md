---
layout:     post
title:      "IP地址归属查询"
subtitle:   "JAVA项目中如何获取IP地址？又怎么获取IP对应的省/城市？不妨来这一看..."
date:       2022-07-08 12:00:00
author:     "LiuYang"
catalog:    false
header-style: text
tags:
  - IP
  - JAVA
---

## 获取IP归属地步骤

* 1、java服务端通过HttpServletRequest对象，获取访问用户的IP地址
* 2、通过IP地址，获取对应的归属地



## 一、获取IP地址
>这一步相信对所有的java小伙伴都已在项目中经常应用，用户用浏览器访问网页，浏览器发起http请求时会将IP地址信息
>放如请求头中，我们通过HttpServletRequest对象获取IP。

```

/**
 * 获取IP地址
 */
public String getIpAddress(ServerHttpRequest request) {
        HttpHeaders headers = request.getHeaders();
        String ipAddress = headers.getFirst("X-Forwarded-For");
        if (ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = headers.getFirst("Proxy-Client-IP");
        }
        if (ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = headers.getFirst("WL-Proxy-Client-IP");
        }
        if (ipAddress == null || ipAddress.length() == 0 || "unknown".equalsIgnoreCase(ipAddress)) {
            ipAddress = request.getRemoteAddress().getAddress().getHostAddress();
            if (ipAddress.equals("127.0.0.1") || ipAddress.equals("0:0:0:0:0:0:0:1")) {
                ipAddress = "127.0.0.1";
            }
        }

        // 对于通过多个代理的情况，第一个IP为客户端真实IP，多个IP按照','分割
        if (ipAddress != null && ipAddress.indexOf(",") > 0) {
            ipAddress = ipAddress.split(",")[0];
        }
        return ipAddress;
    }

```


## 二、获取IP归属地
>使用ip2region开源工具,GitHub仓库地址：[https://github.com/lionsoul2014/ip2region](https://github.com/lionsoul2014/ip2region)


* 1、引入maven依赖：

```
<dependency>
    <groupId>org.lionsoul</groupId>
    <artifactId>ip2region</artifactId>
    <version>2.6.4</version>
</dependency>
```
   
* 2、下载ip2region.xdb,该文件相对路径（ ip2region/data/ip2region.xdb ）

* 3、创建工具栏IpUtils，提供参考代码：

```
import org.lionsoul.ip2region.xdb.Searcher;


public class IpUtils {
    /**
     * ip2region.xdb 文件路径
     * TODO 本机实际文件地址
     */
    private static String DP_PATH = "C:\\Users\\Administrator\\Downloads\\ip2region.xdb";

    /**
     * 查询对象
     */
    private static Searcher SEARCHER;

    /**
     * 加载xdb到内存，并创建查询对象
     */
    static {
        try {
            SEARCHER = Searcher.newWithBuffer(Searcher.loadContentFromFile(DP_PATH));
        } catch (Exception e) {
            e.printStackTrace();
            System.out.print("failed to create content cached searcher");
        }
    }

    /**
     * 根据IP获取归属地
     *
     * @param ip
     * @return
     */
    public static String getIpLocation(String ip) {
        String location = "未知地";
        try {
            location = SEARCHER.search(ip);
            if (location != null && !"".equals(location)) {
                location = location.replace("|0", "");
                location = location.replace("0|", "");
                location = location.replace("|", "-");
                return location;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return location;
    }

    public static void main(String[] args) {
        String ip = "117.185.165.31";
        System.out.println(getIpLocation(ip));
        //控制台打印：中国-上海-上海市-移动
    }
}

```

