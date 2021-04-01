# Golang | ip2location介绍

##### 推荐编辑：laocheng

## 一、简介 
很多时候，我们获取了用户ip，但是想知道更多信息，怎么办？使用`ip2location`吧。

这个库，可以从IP地址快速查找国家，地区，城市，纬度，经度，邮政编码，时区，ISP，域名，连接类型，IDD代码，地区代码 等各种信息。

它使用IP2Location.com上提供的基于文件的数据库，该数据库是以ip为key，国家/城市/经纬度等信息为value 的一个映射表。

## 二、快速使用
步骤如下：
> 1. IP2Location.com下载文件数据库到本地
> 2. 加载文件数据库到代码
> 3. 调用函数获取 国家/城市/经纬度 等数据


### 1. 按需函数调用

在项目中，需要从ip中获取国家、地区、经纬度等信息时候，直接调用对应函数，可以获取相应数值。

```go
package main

import (
    "fmt"
    "github.com/ip2location/ip2location-go"
)

func main() {
    ip := "8.8.8.8"

    ip2location.Open("./IP2LOCATION-LITE-DB5.BIN")
    
    country := ip2location.Get_country_long(ip)
    region := ip2location.Get_region(ip)
    city := ip2location.Get_city(ip)
    latitude := ip2location.Get_latitude(ip)
    longitude := ip2location.Get_longitude(ip)

    fmt.Printf("country: %s\n", country.Country_long)
    fmt.Printf("region: %s\n", region.Region)
    fmt.Printf("city: %s\n", city.City)
    fmt.Printf("latitude: %f\n", latitude.Latitude)
    fmt.Printf("longitude: %f\n", longitude.Longitude)

    ip2location.Close()

}
```
输出如下：
```
country: United States of America
region: California
city: Mountain View
latitude: 37.405991
longitude: -122.078514
```

### 2. 全量数据获取函数

在项目中，需要从ip中获取国家、地区、经纬度等信息时候，直接调用对应函数，可以获取相应数值。

```go
package main

import (
    "fmt"
    "github.com/ip2location/ip2location-go"
)

func main() {
    ip := "8.8.8.8"
    
    ip2location.Open("./IP2LOCATION-LITE-DB5.BIN")

    results := ip2location.Get_all(ip)

    fmt.Printf("country_short: %s\n", results.Country_short)
    fmt.Printf("country_long: %s\n", results.Country_long)
    fmt.Printf("region: %s\n", results.Region)
    fmt.Printf("city: %s\n", results.City)
    fmt.Printf("isp: %s\n", results.Isp)
    fmt.Printf("latitude: %f\n", results.Latitude)
    fmt.Printf("longitude: %f\n", results.Longitude)
    fmt.Printf("domain: %s\n", results.Domain)
    fmt.Printf("zipcode: %s\n", results.Zipcode)
    fmt.Printf("timezone: %s\n", results.Timezone)
    fmt.Printf("netspeed: %s\n", results.Netspeed)
    fmt.Printf("iddcode: %s\n", results.Iddcode)
    fmt.Printf("areacode: %s\n", results.Areacode)
    fmt.Printf("weatherstationcode: %s\n", results.Weatherstationcode)
    fmt.Printf("weatherstationname: %s\n", results.Weatherstationname)
    fmt.Printf("mcc: %s\n", results.Mcc)
    fmt.Printf("mnc: %s\n", results.Mnc)
    fmt.Printf("mobilebrand: %s\n", results.Mobilebrand)
    fmt.Printf("elevation: %f\n", results.Elevation)
    fmt.Printf("usagetype: %s\n", results.Usagetype)
    fmt.Printf("api version: %s\n", ip2location.Api_version())

    ip2location.Close()
}
```

### 3. 一些说明
* 如果仅需要查询IPv4地址，请使用IPv4 BIN文件。
* 如果同时需要查询IPv4地址和IPv6地址，请使用IPv6 BIN文件。


## 总结

`ip2location`库的使用非常简单，直接加载文件数据库，调用相关函数即可。
目前很多国家都推行GDPR政策，网站不允许记录ip等隐私信息，那么`ip2location`库就有了巨大的使用空间。

## 参考资料

* [github.com/ip2location/ip2location-go](github.com/ip2location/ip2location-go)
* [IP2Location.com](IP2Location.com)

---

欢迎加入 GOLANG 中国社区：[https://gocn.vip](https://gocn.vip)





















