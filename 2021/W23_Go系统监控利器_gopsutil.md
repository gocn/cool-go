# Go系统监控利器-gopsutil

## 一、简介

### 什么是gopsutil？

要说**gopsutil**就不得不先了解psutil,顾名思义，psutil = process and system utilities，
它是Python的跨平台库，能够轻松实现获取系统运行的进程和系统利用率，包括CPU、内存、磁盘、网络等信息。
而**gopsutil**就是**psutil**的Golang移植版。


### 为什么用gopsutil？

和直接使用syscall调用对应的系统方法相比，gopsutil为我们屏蔽了各个系统间的差异，可移植性非常强。

## 二、快速上手

安装：
```shell
go get github.com/shirou/gopsutil
```
使用：

```go
package main

import (
	"fmt"
	"github.com/shirou/gopsutil/cpu"
	"time"
)

func main() {
	info, _ := cpu.Info()

	per, _ := cpu.Percent(1 * time.Second, true)

	fmt.Printf("CPU Percent: %f\n", per)

	fmt.Println(info)
}


```

输出：
```
CPU Percent: [4.040404 4.000000 5.050505 6.930693]
[{"cpu":0,"vendorId":"AuthenticAMD","family":"23","model":"49","stepping":0,"physicalId":"0","coreId":"0","cores":1,"modelName":"AMD EPYC 7K62 48-Core Processor","mhz":2595.124,"cacheSize":512,"flags":["fpu","vme","de","pse","tsc","msr","pae","mce","cx8","apic","sep","mtrr","pge","mca","cmov","pat","pse36","clflush","mmx","fxsr","sse","sse2","ht","syscall","nx","mmxext","fxsr_opt","pdpe1gb","rdtscp","lm","rep_good","nopl","cpuid","extd_apicid","tsc_known_freq","pni","pclmulqdq","ssse3","fma","cx16","sse4_1","sse4_2","x2apic","movbe","popcnt","aes","xsave","avx","f16c","rdrand","hypervisor","lahf_lm","cmp_legacy","cr8_legacy","abm","sse4a","misalignsse","3dnowprefetch","osvw","topoext","ibpb","vmmcall","fsgsbase","bmi1","avx2","smep","bmi2","rdseed","adx","smap","clflushopt","sha_ni","xsaveopt","xsavec","xgetbv1","arat"],"microcode":"0x1000065"} {"cpu":1,"vendorId":"AuthenticAMD","family":"23","model":"49","stepping":0,"physicalId":"0","coreId":"1","cores":1,"modelName":"AMD EPYC 7K62 48-Core Processor","mhz":2595.124,"cacheSize":512,"flags":["fpu","vme","de","pse","tsc","msr","pae","mce","cx8","apic","sep","mtrr","pge","mca","cmov","pat","pse36","clflush","mmx","fxsr","sse","sse2","ht","syscall","nx","mmxext","fxsr_opt","pdpe1gb","rdtscp","lm","rep_good","nopl","cpuid","extd_apicid","tsc_known_freq","pni","pclmulqdq","ssse3","fma","cx16","sse4_1","sse4_2","x2apic","movbe","popcnt","aes","xsave","avx","f16c","rdrand","hypervisor","lahf_lm","cmp_legacy","cr8_legacy","abm","sse4a","misalignsse","3dnowprefetch","osvw","topoext","ibpb","vmmcall","fsgsbase","bmi1","avx2","smep","bmi2","rdseed","adx","smap","clflushopt","sha_ni","xsaveopt","xsavec","xgetbv1","arat"],"microcode":"0x1000065"} {"cpu":2,"vendorId":"AuthenticAMD","family":"23","model":"49","stepping":0,"physicalId":"0","coreId":"2","cores":1,"modelName":"AMD EPYC 7K62 48-Core Processor","mhz":2595.124,"cacheSize":512,"flags":["fpu","vme","de","pse","tsc","msr","pae","mce","cx8","apic","sep","mtrr","pge","mca","cmov","pat","pse36","clflush","mmx","fxsr","sse","sse2","ht","syscall","nx","mmxext","fxsr_opt","pdpe1gb","rdtscp","lm","rep_good","nopl","cpuid","extd_apicid","tsc_known_freq","pni","pclmulqdq","ssse3","fma","cx16","sse4_1","sse4_2","x2apic","movbe","popcnt","aes","xsave","avx","f16c","rdrand","hypervisor","lahf_lm","cmp_legacy","cr8_legacy","abm","sse4a","misalignsse","3dnowprefetch","osvw","topoext","ibpb","vmmcall","fsgsbase","bmi1","avx2","smep","bmi2","rdseed","adx","smap","clflushopt","sha_ni","xsaveopt","xsavec","xgetbv1","arat"],"microcode":"0x1000065"} {"cpu":3,"vendorId":"AuthenticAMD","family":"23","model":"49","stepping":0,"physicalId":"0","coreId":"3","cores":1,"modelName":"AMD EPYC 7K62 48-Core Processor","mhz":2595.124,"cacheSize":512,"flags":["fpu","vme","de","pse","tsc","msr","pae","mce","cx8","apic","sep","mtrr","pge","mca","cmov","pat","pse36","clflush","mmx","fxsr","sse","sse2","ht","syscall","nx","mmxext","fxsr_opt","pdpe1gb","rdtscp","lm","rep_good","nopl","cpuid","extd_apicid","tsc_known_freq","pni","pclmulqdq","ssse3","fma","cx16","sse4_1","sse4_2","x2apic","movbe","popcnt","aes","xsave","avx","f16c","rdrand","hypervisor","lahf_lm","cmp_legacy","cr8_legacy","abm","sse4a","misalignsse","3dnowprefetch","osvw","topoext","ibpb","vmmcall","fsgsbase","bmi1","avx2","smep","bmi2","rdseed","adx","smap","clflushopt","sha_ni","xsaveopt","xsavec","xgetbv1","arat"],"microcode":"0x1000065"}]
```



## 三、分工明确

### gopsutil将不同的功能划分到不同的子包中：

主要为cpu,disk,docker,host,mem,net,process,winservices这几个。
想要使用对应的功能，要导入对应的子包。例如，上面代码中，我们要获取CPU信息，导入的是cpu子包。
上述样例中，我们获取到了每个cpu的占用率和所有cpu的详细信息。


## 闲言

最近正在写一个Golang实现性能监控的demo，之后还会写这方面的介绍或者对比。


## 参考文档：

- https://github.com/shirou/gopsutil

欢迎加入我们GOLANG中国社区：https://gocn.vip/
