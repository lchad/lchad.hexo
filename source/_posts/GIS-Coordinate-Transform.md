
---
title: GIS坐标转换
date: 2017-04-93 14:29
categories: GIS
toc: true
---
最近在做一个千寻FindM高精度定位的项目,由于对GIS的知识储备十分有限,其中坐标转换的问题花了我不少时间,所以在这里总结一下.

## 经纬度的格式
经纬的表示方法有三种:
- 度 ddd.ddddd°
- 度分  dd°mm.mmm′
- 度分秒 ddd°mm′ss″ 

我们最常使用的就是第一种,度格式,如`(31.220018, 121.351505)`
但是我的项目中,需要从硬件(Gmouse)读取出来经纬度,然后把它正确的显示在高德地图上,以此来测试定位的精确度,但是我得到的结果却是`(3113.33126, 12120.85937)`这样的度分格式,所以需要进行转换,度分格式的意义如下:
`(3113.33126, 12120.85937) ==> (31°13.33126′, 121°20.85937′)`
小数点左边的高两位代表度的整数,高两位后面的(如13.33126)代表分的大小.

度分格式转换为度格式的公式为:
``
abcd.efghd ==> ab° + cd.efghd′ ==> ab° + (cd.dfghd / 60)°
``
度分秒格式的转换和度分格式几乎一样:
`(313000, 1233000) ==> (31°30′00″, 123°30′00″)`

度分秒格式转换为度的公式为:
``
abcdefg ==> abc° + cd′ + dfg″ ==> ab° + (cd / 60)° + (def / 60 / 60)°
``

## 经纬度的坐标系
我们常用的经纬度的坐标系有如下几种:
- WGS84 (世界通用标准)
- GCJ02 (天朝标准)
- BD09 (百度标准)

WGS84坐标是全世界公认的坐标体系,使用最为广泛,Google Map使用的就是这个坐标系;GCJ02是中国特色主义标准,声称为了国家安全,对WGS84标准坐标系做一些offset(妈的智障),大多数中国地图提供商都是使用的这个坐标系;BD09是百度的坐标系,其实也是基于GCJ02,相当与又做了一次offset.
他们的转换关系可以用如下代码来描述:
```
    public static double pi = 3.1415926535897932384626;
    public static double a = 6378245.0;
    public static double ee = 0.00669342162296594323;

    /**
     * 地球坐标系 (WGS84) 与中国坐标系 (GCJ-02) 的转换算法 将 WGS84 坐标转换成 GCJ-02 坐标
     */
    public static double[] wgs84_To_Gcj02(double lat, double lon) {
        if (lon < 72.004 || lon > 137.8347) {
            return null;
        }
        if (lat < 0.8293 || lat > 55.8271) {
            return null;
        }
        double dLat = transformLat(lon - 105.0, lat - 35.0);
        double dLon = transformLon(lon - 105.0, lat - 35.0);
        double radLat = lat / 180.0 * pi;
        double magic = Math.sin(radLat);
        magic = 1 - ee * magic * magic;
        double sqrtMagic = Math.sqrt(magic);
        dLat = (dLat * 180.0) / ((a * (1 - ee)) / (magic * sqrtMagic) * pi);
        dLon = (dLon * 180.0) / (a / sqrtMagic * Math.cos(radLat) * pi);
        double mgLat = lat + dLat;
        double mgLon = lon + dLon;
        return new double[]{mgLon, mgLat};
    }

    /**
     * 中国坐标系 (GCJ-02) 与百度坐标系 (BD-09) 的转换算法 将 GCJ-02 坐标转换成 BD-09 坐标
     */
    public static double[] gcj02_To_Bd09(double gg_lon, double gg_lat) {
        double z = Math.sqrt(gg_lon * gg_lon + gg_lat * gg_lat) + 0.00002 * Math.sin(gg_lat * pi);
        double theta = Math.atan2(gg_lat, gg_lon) + 0.000003 * Math.cos(gg_lon * pi);
        double bd_lon = z * Math.cos(theta) + 0.0065;
        double bd_lat = z * Math.sin(theta) + 0.006;
        return new double[]{bd_lon, bd_lat};
    }

    /**
     * 地球坐标系 (GCJ-02) 与百度坐标系 (BD-09) 的转换算法 将 GCJ-02 坐标转换成 BD-09 坐标
     */
    public static double[] wgs84_To_Bd09(double lon, double lat) {
        if (lon < 72.004 || lon > 137.8347) {
            return null;
        }
        if (lat < 0.8293 || lat > 55.8271) {
            return null;
        }
        double dLat = transformLat(lon - 105.0, lat - 35.0);
        double dLon = transformLon(lon - 105.0, lat - 35.0);
        double radLat = lat / 180.0 * pi;
        double magic = Math.sin(radLat);
        magic = 1 - ee * magic * magic;
        double sqrtMagic = Math.sqrt(magic);
        dLat = (dLat * 180.0) / ((a * (1 - ee)) / (magic * sqrtMagic) * pi);
        dLon = (dLon * 180.0) / (a / sqrtMagic * Math.cos(radLat) * pi);
        double mgLat = lat + dLat;
        double mgLon = lon + dLon;
        return gcj02_To_Bd09(mgLon, mgLat);
    }

    private static double transformLat(double x, double y) {
        double ret = -100.0 + 2.0 * x + 3.0 * y + 0.2 * y * y + 0.1 * x * y
                + 0.2 * Math.sqrt(Math.abs(x));
        ret += (20.0 * Math.sin(6.0 * x * pi) + 20.0 * Math.sin(2.0 * x * pi)) * 2.0 / 3.0;
        ret += (20.0 * Math.sin(y * pi) + 40.0 * Math.sin(y / 3.0 * pi)) * 2.0 / 3.0;
        ret += (160.0 * Math.sin(y / 12.0 * pi) + 320 * Math.sin(y * pi / 30.0)) * 2.0 / 3.0;
        return ret;
    }

    private static double transformLon(double x, double y) {
        double ret = 300.0 + x + 2.0 * y + 0.1 * x * x + 0.1 * x * y + 0.1
                * Math.sqrt(Math.abs(x));
        ret += (20.0 * Math.sin(6.0 * x * pi) + 20.0 * Math.sin(2.0 * x * pi)) * 2.0 / 3.0;
        ret += (20.0 * Math.sin(x * pi) + 40.0 * Math.sin(x / 3.0 * pi)) * 2.0 / 3.0;
        ret += (150.0 * Math.sin(x / 12.0 * pi) + 300.0 * Math.sin(x / 30.0 * pi)) * 2.0 / 3.0;
        return ret;
    }
```

示例代码如下:
[GitHub](https://github.com/lchad/GnnsSample)