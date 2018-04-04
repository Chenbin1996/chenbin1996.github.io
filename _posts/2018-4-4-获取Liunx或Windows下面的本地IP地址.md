---
layout:     post
title:      获取Liunx或者Windows的本地IP地址
subtitle:   Java中获取本地IP地址的方式有很多种，但分系统，这些做一个简单工具类判断
date:       2018-04-04
author:     如漩涡
header-img: img/post-bg-debug.png
catalog: true
tags:
    - Java
---

<p>封装了一个工具类，这个工具类很简单，就是获取不同系统下的IP地址，直接Copy过去就能使用</p>

```Java
import java.net.InetAddress;
import java.net.NetworkInterface;
import java.util.Enumeration;
/**
 * 自制工具类
 * 用于获取windows与linux系统下的ip地址
 * @author chenbin on 2018/02/28
 * @version 3.0.0
 */
public class IpToolUtil {

    public static boolean isWindowOS() {
        boolean isWindowOS = false;
        String osName = System.getProperty("os.name");
        String windows = "windows";
        if (osName.toLowerCase().indexOf(windows) > -1) {
            isWindowOS = true;
        }
        return isWindowOS;
    }

    public static InetAddress getInetAddress() {
        InetAddress inetAddress = null;
        try {
            //如果是windows操作系统
            if (isWindowOS()) {
                inetAddress = InetAddress.getLocalHost();
            } else {
                boolean bFindIP = false;
                //定义一个内容都是NetworkInterface的枚举对象
                Enumeration<NetworkInterface> netInterfaces = (Enumeration<NetworkInterface>)
                        NetworkInterface.getNetworkInterfaces();
                //如果枚举对象里面还有内容(NetworkInterface)
                while (netInterfaces.hasMoreElements()) {
                    if (bFindIP) {
                        break;
                    }
                    //获取下一个内容(NetworkInterface)
                    NetworkInterface ni = (NetworkInterface) netInterfaces.nextElement();
                    //----------特定情况，可以考虑用ni.getName判断
                    //遍历所有IP
                    Enumeration<InetAddress> ips = ni.getInetAddresses();
                    while (ips.hasMoreElements()) {
                        inetAddress = (InetAddress) ips.nextElement();
                        //属于本地地址->不是回环地址->地址里面没有:号
                        if (inetAddress.isSiteLocalAddress()        
                                && !inetAddress.isLoopbackAddress() 
                                && inetAddress.getHostAddress().indexOf(":") == -1) {
                            //找到了地址
                            bFindIP = true;
                            //退出循环
                            break;              
                        }
                    }
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return inetAddress;
    }

    public static String getLocalIP() {
        return getInetAddress().getHostAddress();
    }
}
```