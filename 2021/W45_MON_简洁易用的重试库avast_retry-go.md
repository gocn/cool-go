# 简洁易用的重试库avast/retry-go

## 推荐理由

在分布式环境中，常常会发生因网络抖动或资源响应缓慢或暂时不可用，进而导致对服务或资源远程调用失败的情况，这种场景下，可以通过重试策略提高系统最终的稳定性。但如果在高并发的场景下，简单的for循环重试会导致流量放大，使系统故障进一步恶化。本次介绍一款简洁易用的重试库来实现优雅的重试。

## 功能介绍

retry-go支持以下功能：
1. 两次调用之间等待固定的时间间隔；
2. 两次调用之间等待随机的时间间隔，最大值时间间隔可配置；
3. 两次调用等待时间间隔，按照指数退避；
4. 支持上面三种类型的策略相结合的方式设置两次调用之间的时间间隔；
5. 可根据是否发生特定的error来重试；
6. 支持设置重试回调函数。

## 使用指南

### 安装

```shell
go get github.com/avast/retry-go
```

### 代码示例

下面一个简单的例子：

```golang
package main

import (
	"fmt"
	"time"

	"github.com/avast/retry-go"
	"github.com/pkg/errors"
)

func main() {

	err := retry.Do(
		func() error {
			return errors.New("test")
		},
		// 设置重试次数为3次
		retry.Attempts(3),
		// 设置最大delay时间为100毫秒
		retry.MaxDelay(100*time.Millisecond),
		// 设置最大随机delay时间为100毫秒
		retry.MaxJitter(100*time.Millisecond),
		// 设置重试间隔算法为随机+指数退避
		retry.DelayType(retry.CombineDelay(retry.BackOffDelay, retry.RandomDelay)),
		// 回调函数
		retry.OnRetry(func(n uint, err error) {
			fmt.Println("OnRetry: ", n, err)
		}),
		// 当error为test时继续重试，否则退出重试
		retry.RetryIf(func(e error) bool {
			return e.Error() == "test"
		}),
	)

	fmt.Println(err)

}
```

retry-go库的使用相对比较简单，能够满足不同场景下的重试需求。

## 总结

retry-go库针对重试间隔时间，提供了多种策略，并且可以多个策略组合使用，可以满足大部分的重试场景。如果你的业务代码中还有简单for循环重试的逻辑，推荐用retry-go库来重构。

## 参考资料

1. [https://github.com/avast/retry-go](https://github.com/avast/retry-go)