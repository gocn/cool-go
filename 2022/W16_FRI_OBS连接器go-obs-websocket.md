# OBS连接器go-obs-websocket
## 推荐理由
互联网的兴起带动了直播行业的火热，除了少数直播网站有自己的推流工具之外，OBS是主流的推流工具，广泛应用在直转播技术之上。

## 简介
go-obs-websocket是一个与OBS进行websocket通信的连接库，具备调用大部分OBS功能的接口，在互动直播和智能转播技术上广泛应用。

## 快速开始
### 安装
```shell 
go get github.com/christopher-dG/go-obs-websocket
```

### obs简介
#### 界面
![[obs.png]]
点击<工具>可以设置websocket的参数，默认是监听本地的4444端口

### 建立连接
```go
	import obsws "github.com/christopher-dG/go-obs-websocket"

	c := obsws.Client{Host: "localhost", Port: 4444}
	if err := c.Connect(); err != nil {
		log.Fatal(err)
	}
	defer c.Disconnect()
```

### 发送请求
```go
	// Send and receive a request asynchronously.
	req := obsws.NewGetStreamingStatusRequest()
	if err := req.Send(c); err != nil {
		log.Fatal(err)
	}

	// This will block until the response comes (potentially forever).
	resp, err := req.Receive()
	if err != nil {
		log.Fatal(err)
	}
	log.Println("streaming:", resp.Streaming)
```
+ 创建请求，这里创建的是流状态请求
+ 发送请求
+ 获取请求结果
这里的请求会阻塞当前协程，当OBS正处于推流时，resp.Steaming值为true，反之为false。

*go-obs-websocket*  提供了众多请求接口，可以使用如下命令获取
```shell
#linux
go doc github.com/christopher-dG/go-obs-websocket |grep "Request " |grep type

#windows
go doc github.com/christopher-dG/go-obs-websocket |findstr "Request " | findstr type
```

接下来我们挑几个常见的来看看使用案例
#### 开始/停止推流
```go
	req := obsws.NewStartStopStreamingRequest()
	if err := req.Send(c); err != nil {
		log.Fatal(err)
	}

	resp, err := req.Receive()
	if err != nil {
		log.Fatal(err)
	}
	log.Println("streaming:", resp.Status())
```
#### 开始录制
```go
	req := obsws.NewStartRecordingRequest()
	if err := req.Send(c); err != nil {
		log.Fatal(err)
	}

	resp, err := req.Receive()
	if err != nil {
		log.Fatal(err)
	}
	log.Println("streaming:", resp.Status())
```
#### 停止录制
```go
	req := obsws.NewStopRecordingRequest()
	if err := req.Send(c); err != nil {
		log.Fatal(err)
	}

	resp, err := req.Receive()
	if err != nil {
		log.Fatal(err)
	}
	log.Println("streaming:", resp.Status())
```
#### 转换场景
```go
	req := obsws.NewSetCurrentSceneRequest("scene1")
	if err := req.Send(c); err != nil {
		log.Fatal(err)
	}

	resp, err := req.Receive()
	if err != nil {
		log.Fatal(err)
	}
	log.Println("streaming:", resp.Status())
```

### 处理响应
```go
    c.AddEventHandler("SwitchScenes", func(e obsws.Event) {
        // Make sure to assert the actual event type.
        log.Println("new scene:", e.(obsws.SwitchScenesEvent).SceneName)
    })
```

注册钩子函数，响应OBS的信号。
一共可注册的钩子函数key如下：

```
"SwitchScenes"                  场景切换
"ScenesChanged"                 场景变化
"SceneCollectionChanged"        场景来源变化
"SceneCollectionListChanged"    场景来源列表变化
"SwitchTransition"              转场
"TransitionListChanged"         转场列表变化
"TransitionDurationChanged"     转场时长变化
"TransitionBegin"               开始转场
"ProfileChanged"                配置变化
"ProfileListChanged"            配置文件列表变化
"StreamStarting"                推流开始启动
"StreamStarted"                 推流启动完成
"StreamStopping"                推流正在停止
"StreamStopped"                 推流已暂停
"StreamStatus"                  推流状态
"RecordingStarting"             
"RecordingStarted"              
"RecordingStopping"             
"RecordingStopped"              
"RecordingPaused"               
"RecordingResumed"              
"ReplayStarting"                回放
"ReplayStarted"                 
"ReplayStopping"                
"ReplayStopped"                 
"Exiting"                      退出 
"Heartbeat"                    心跳 
"BroadcastCustomMessage"       广播消息 
"SourceCreated"                来源创建
"SourceDestroyed"              来源销毁
"SourceVolumeChanged"          来源音频改变
"SourceMuteStateChanged"       音量改变
"SourceAudioSyncOffsetChanged" 
"SourceAudioMixersChanged"      
"SourceRenamed"                 
"SourceFilterAdded"            滤波添加 
"SourceFilterRemoved"          滤波移除
"SourceFilterVisibilityChanged" 滤波可见性改变
"SourceFiltersReordered"        
"SourceOrderChanged"           来源顺序变换 
"SceneItemAdded"               场景元素添加
"SceneItemRemoved"             场景元素移除 
"SceneItemVisibilityChanged"    
"SceneItemTransformChanged"     
"SceneItemSelected"             
"SceneItemDeselected"           
"PreviewSceneChanged"           
"StudioModeSwitched"            
```


## 总结
OBS是使用最广的推流工具，利用go-obs-websocket可以很方便的操作大部分功能，目前在日常操作中只有创建场景的接口没有找到。比如我们可以做一系列的转场，在直播推流过程中监听事件自动触发，或者做一些特效能随着转场渐入渐出。祝大家玩的开心！！

## 参考资料
+ [go-obs-websocket](https://github.com/christopher-dG/go-obs-websocket)
+ [obs-websocket](https://github.com/obsproject/obs-websocket)

