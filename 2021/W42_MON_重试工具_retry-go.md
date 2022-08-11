# 重试工具 — retry-go

## 简介

在微服务架构中，通常会有很多的小服务，小服务之间存在大量 RPC 调用，但时常因为网络抖动等原因，造成请求失败，这时候使用重试机制可以提高请求的最终成功率，减少故障影响，让系统运行更稳定。[retry-go](https://github.com/avast/retry-go) 是一个功能比较完善的 golang 重试库。

## 如何使用：

`retry-go`的使用非常简单，直接使用 `Do`方法即可。如下是一个发起 HTTP Get 请求的重试示例 :

```go
url := "https://gocn.vip"
var body []byte

err := retry.Do(
	func() error {
		resp, err := http.Get(url)
		if err != nil {
			return err
		}
		defer resp.Body.Close()
		body, err = ioutil.ReadAll(resp.Body)
		if err != nil {
			return err
		}

		return nil
	},
)

fmt.Println(body)

```

调用时，有一些可选的配置项：

-  ***attempts*** 最大重试次数
-  ***delay*** 重试延迟时间
-  ***maxDelay*** 最大重试延迟时间，选择指数退避策略时，该配置会限制等待时间上限
-  ***maxJitter*** 随机退避策略的最大等待时间
-  ***onRetry*** 每次重试时进行的一次回调
-  ***retryIf*** 重试时的一个条件判断
-  ***delayType*** 退避策略类型
-  ***lastErrorOnly*** 是否只返回上次重试的错误


### BackOff 退避策略

对于一些暂时性的错误，如网络抖动等，立即重试可能还是会失败，通常等待一小会儿再重试的话成功率会较高，并且这种策略也可以打散上游重试的时间，避免同时重试而导致的瞬间流量高峰。决定等待多久之后再重试的方法叫做退避策略。`retry-go` 实现了以下几个退避策略：

#### func BackOffDelay

```
func BackOffDelay(n uint, _ error, config *Config) time.Duration
```

BackOffDelay 提供一个指数避退策略，连续重试时，每次等待时间都是前一次的 2 倍。

#### func FixedDelay

```
func FixedDelay(_ uint, _ error, config *Config) time.Duration
```

FixedDelay 在每次重试时，等待一个固定延迟时间。

#### func RandomDelay

```
func RandomDelay(_ uint, _ error, config *Config) time.Duration
```

RandomDelay 在 0 - config.maxJitter 内随机等待一个时间后重试。

#### func CombineDelay

```
func CombineDelay(delays ...DelayTypeFunc) DelayTypeFunc
```

CombineDelay  提供结合多种策略实现一个新策略的能力。

`retry-go`默认的退避策略为  `BackOffDelay`和`RandomDelay`结合的方式，即在指数递增的同时，加一个随机时间。

### 自定义的延时策略

下面是一个官方给出的例子，当请求的响应有`Retry-After`头时，使用该值去进行等待，其他情况按照BackOffDelay策略进行延时等待。

```go
var _ error = (*RetriableError)(nil)

func test2(){
	var body []byte

	err := retry.Do(
		func() error {
			resp, err := http.Get("URL")

			if err == nil {
				defer func() {
					if err := resp.Body.Close(); err != nil {
						panic(err)
					}
				}()
				body, err = ioutil.ReadAll(resp.Body)
				if resp.StatusCode != 200 {
					err = fmt.Errorf("HTTP %d: %s", resp.StatusCode, string(body))
					if resp.StatusCode == http.StatusTooManyRequests {
						// check Retry-After header if it contains seconds to wait for the next retry
						if retryAfter, e := strconv.ParseInt(resp.Header.Get("Retry-After"), 10, 32); e == nil {
							// the server returns 0 to inform that the operation cannot be retried
							if retryAfter <= 0 {
								return retry.Unrecoverable(err)
							}
							return &RetriableError{
								Err:        err,
								RetryAfter: time.Duration(retryAfter) * time.Second,
							}
						}
						// A real implementation should also try to http.Parse the retryAfter response header
						// to conform with HTTP specification. Herein we know here that we return only seconds.
					}
				}
			}

			return err
		},
		retry.DelayType(func(n uint, err error, config *retry.Config) time.Duration {
			fmt.Println("Server fails with: " + err.Error())
			if retriable, ok := err.(*RetriableError); ok {
				fmt.Printf("Client follows server recommendation to retry after %v\n", retriable.RetryAfter)
				return retriable.RetryAfter
			}
			// apply a default exponential back off strategy
			return retry.BackOffDelay(n, err, config)
		}),
	)

	fmt.Println("Server responds with: " + string(body))
}
```


## 总结

重试可以提升服务调用的成功率，但重试时也要警惕由此带来的放大故障的风险。选择合适的退避策略，控制放大效应，才能优雅的提升服务的稳定性。

## Reference

[如何优雅地重试-InfoQ](https://www.infoq.cn/article/5fboevkal0gvgvgeac4z)

[[译\] 重试、超时和退避 | nettee 的 blog](https://nettee.github.io/posts/2019/Retries-Timeouts-and-Backoff/)

