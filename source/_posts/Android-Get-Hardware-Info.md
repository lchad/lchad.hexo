---
title: 获取手机的相关硬件信息
date: 2015-05-09 00:07
categories: Android
toc: true
cover: /img/android.jpg
---
今天在QQ群里聊天,一个哥们在某宝买到了一个运行内存16G的手机,当时我就吓尿了,所以有了写个程序把这个手机的实际内存读出来的想法,于是就有了今天这篇博客.
<!--more-->
所有的信息项如下图所示.(由于我的测试机没有插手机卡,所以有的信息会显示为空)
![](http://upload-images.jianshu.io/upload_images/174711-3cb15053398447ee?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以下就是代码:


```
package com.liu.chad.practicesqlite;  
  
import android.app.ActivityManager;  
import android.content.Context;  
import android.os.Bundle;  
import android.support.v7.app.ActionBarActivity;  
import android.telephony.TelephonyManager;  
import android.text.format.Formatter;  
import android.util.Log;  
import android.widget.TextView;  
  
import java.io.BufferedReader;  
import java.io.FileNotFoundException;  
import java.io.FileReader;  
import java.io.IOException;  
import java.io.InputStream;  
  
public class MainActivity extends ActionBarActivity {  
  
    private TextView mTextView;  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
  
        mTextView = (TextView) findViewById(R.id.textViewId);  
        getPhoneInfo();  
    }  
  
    /** 
     * 获取手机信息 
     */  
    public void getPhoneInfo() {  
        TelephonyManager tm = (TelephonyManager) this.getSystemService(TELEPHONY_SERVICE);  
        String mtyb = android.os.Build.BRAND;// 手机品牌  
        String mtype = android.os.Build.MODEL; // 手机型号  
        String imei = tm.getDeviceId();  
        String imsi = tm.getSubscriberId();  
        String numer = tm.getLine1Number(); // 手机号码  
        String serviceName = tm.getSimOperatorName(); // 运营商  
        mTextView.setText("品牌: " + mtyb + "\n" + "型号: " + mtype + "\n" + "版本: Android "  
                + android.os.Build.VERSION.RELEASE + "\n" + "IMEI: " + imei  
                + "\n" + "IMSI: " + imsi + "\n" + "手机号码: " + numer + "\n"  
                + "运营商: " + serviceName + "\n");  
        mTextView.append("总内存: " + getTotalMemory() + "\n");  
        mTextView.append("当前可用内存: " + getAvailMemory() + "\n");  
        mTextView.append("CPU名字: " + getCpuName() + "\n");  
        mTextView.append("CPU最大频率: " + getMaxCpuFreq() + "\n");  
        mTextView.append("CPU最小频率: " + getMinCpuFreq() + "\n");  
        mTextView.append("CPU当前频率: " + getCurCpuFreq() + "\n");  
    }  
  
    /** 
     * 获取手机内存大小 
     * 
     * @return 
     */  
    private String getTotalMemory() {  
        String str1 = "/proc/meminfo";// 系统内存信息文件  
        String str2;  
        String[] arrayOfString;  
        long initial_memory = 0;  
        try {  
            FileReader localFileReader = new FileReader(str1);  
            BufferedReader localBufferedReader = new BufferedReader(localFileReader, 8192);  
            str2 = localBufferedReader.readLine();// 读取meminfo第一行，系统总内存大小  
  
            arrayOfString = str2.split("\\s+");  
            for (String num : arrayOfString) {  
                Log.i(str2, num + "\t");  
            }  
  
            initial_memory = Integer.valueOf(arrayOfString[1]).intValue() * 1024;// 获得系统总内存，单位是KB，乘以1024转换为Byte  
            localBufferedReader.close();  
  
        } catch (IOException e) {  
        }  
        return Formatter.formatFileSize(getBaseContext(), initial_memory);// Byte转换为KB或者MB，内存大小规格化  
    }  
  
    /** 
     * 获取当前可用内存大小 
     * 
     * @return 
     */  
    private String getAvailMemory() {  
        ActivityManager am = (ActivityManager) getSystemService(Context.ACTIVITY_SERVICE);  
        ActivityManager.MemoryInfo mi = new ActivityManager.MemoryInfo();  
        am.getMemoryInfo(mi);  
        return Formatter.formatFileSize(getBaseContext(), mi.availMem);  
    }  
  
    public static String getMaxCpuFreq() {  
        String result = "";  
        ProcessBuilder cmd;  
        try {  
            String[] args = {"/system/bin/cat",  
                    "/sys/devices/system/cpu/cpu0/cpufreq/cpuinfo_max_freq"};  
            cmd = new ProcessBuilder(args);  
            Process process = cmd.start();  
            InputStream in = process.getInputStream();  
            byte[] re = new byte[24];  
            while (in.read(re) != -1) {  
                result = result + new String(re);  
            }  
            in.close();  
        } catch (IOException ex) {  
            ex.printStackTrace();  
            result = "N/A";  
        }  
        return result.trim() + "Hz";  
    }  
  
    // 获取CPU最小频率（单位KHZ）  
  
    public static String getMinCpuFreq() {  
        String result = "";  
        ProcessBuilder cmd;  
        try {  
            String[] args = {"/system/bin/cat",  
                    "/sys/devices/system/cpu/cpu0/cpufreq/cpuinfo_min_freq"};  
            cmd = new ProcessBuilder(args);  
            Process process = cmd.start();  
            InputStream in = process.getInputStream();  
            byte[] re = new byte[24];  
            while (in.read(re) != -1) {  
                result = result + new String(re);  
            }  
            in.close();  
        } catch (IOException ex) {  
            ex.printStackTrace();  
            result = "N/A";  
        }  
        return result.trim() + "Hz";  
    }  
  
    // 实时获取CPU当前频率（单位KHZ）  
  
    public static String getCurCpuFreq() {  
        String result = "N/A";  
        try {  
            FileReader fr = new FileReader(  
                    "/sys/devices/system/cpu/cpu0/cpufreq/scaling_cur_freq");  
            BufferedReader br = new BufferedReader(fr);  
            String text = br.readLine();  
            result = text.trim() + "Hz";  
        } catch (FileNotFoundException e) {  
            e.printStackTrace();  
        } catch (IOException e) {  
            e.printStackTrace();  
        }  
        return result;  
    }  
  
    public static String getCpuName() {  
        try {  
            FileReader fr = new FileReader("/proc/cpuinfo");  
            BufferedReader br = new BufferedReader(fr);  
            String text = br.readLine();  
            String[] array = text.split(":\\s+", 2);  
            for (int i = 0; i < array.length; i++) {  
            }  
            return array[1];  
        } catch (FileNotFoundException e) {  
            e.printStackTrace();  
        } catch (IOException e) {  
            e.printStackTrace();  
        }  
        return null;  
    }  
}  
```

布局文件就是一个TextView,我就不在这里贴出来了.