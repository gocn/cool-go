# 灵活的Go http client库-Sling

## 推荐理由

项目开发中，发送http请求的场景，推荐使用Sling库。Sling本身是基于net/http来处理发送请求，同时做了较好的封装，既可以利用net/http的一些特性（如：httptrace)，同时又不必关心net/http库的一些琐碎细节。

## 功能介绍

Sling支持以下主要的功能：

* 支持GET/POST/PUT/PATCH/DELETE/HEAD
* 基于Base/Path可以扩展和复用Sling
* query参数可以用结构体来Encode
* Request Body支持form和json
* 可将Json格式的Response直接Decode到定义好的结构体
* 可扩展Response的Decoder，以及Doer。

## 使用指南

1. Sling对http请求的要素method、baseUrl、Path、query、body、request、response等做了封装，基本使用可以参考<https://github.com/dghubble/sling> 上的示例代码。

2. 可以通过实现ResponseDecoder和Doer的接口，来定制响应的decoder和发送请求的具体实现。

3. 利用Sling灵活扩展优势的提高业务代码复用，可以参考下面的示例：

```go
const baseURL = "https://api.github.com/"

// Github Issue (abbreviated)
type Issue struct {
    Title  string `json:"title"`
    Body   string `json:"body"`
}

type IssueService struct {
    sling *sling.Sling
}

func NewIssueService(httpClient *http.Client) *IssueService {
    return &IssueService{
        sling: sling.New().Client(httpClient).Base(baseURL),
    }
}

func (s *IssueService) ListByRepo(owner, repo string, params *IssueListParams) ([]Issue, *http.Response, error) {
    issues := new([]Issue)
    githubError := new(GithubError)
    path := fmt.Sprintf("repos/%s/%s/issues", owner, repo)
    // 注意此处一定要调用New方法来clone一个sling实例
    resp, err := s.sling.New().Get(path).QueryStruct(params).Receive(issues, githubError)
    if err == nil {
        err = githubError
    }
    return *issues, resp, err
}
```

## 总结

## 参考资料

1. [https://github.com/dghubble/sling](https://github.com/dghubble/sling)
