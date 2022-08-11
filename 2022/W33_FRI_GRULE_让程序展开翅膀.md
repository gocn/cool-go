# GRule 让GO语言展开翅膀
一直以来脚本语言的性能和静态语言的热更新都是各类语言逃不开的话题，为此语言宛如八仙过海，各显神通，相互取长补短。今天介绍的将是自创脚本语言无缝连接Go语言的一个库，名曰：grule-rule-engine。
## 前言
grule-rule-engine 是基于antlr4生成的语法词法解析库形成的。antlr可以对规则进行词法，语法解析，对成功解析的语法树提供接口进行数据处理和输出。grule-rule-engine根据自己定义的合理规则，基于antlr生成的接口对字符串进行语法解析和求值，从而实现类似逻辑的热更新。接下来，让我们来一睹尊容，顺便提一下哔哩哔哩也开源了一套类似的规则bilibili/gengine。以下我会以脚本来代表待解析的文本。

## 安装
```go
go get github.com/hyperjumptech/grule-rule-engine
```
## 初识
```
rule  <rule_name> <rule_description>
   <attribute> <value> {
   when
      <conditions>

   then
      <actions>
}
```
脚本区域分为：
+ rule_name: 脚本名称 
+ rule_description: 脚本描述
+ attribute value: 通常会写入权重值: salience 5
+ conditions: 脚本运行的条件，满足条件的脚本将有机会执行
+ actions: 脚本执行的内容

### 脚本例子
```
rule  example1  "example1-rule"  salience 1 {
when
(Fact.Distance > 5000  &&   Fact.Duration > 120) && (Fact.Result == false)
Then
   Fact.NetAmount=143.320007;
   Fact.SetResult(true);
}
```
可以看到脚本是可以引用对象的成员和调用对象方法的。

上面的脚本解释出来的意义就是：
当Fact的距离大于5000的单位，时长超过120并且Result值为false时，将NetAmount赋值并且设置Result为true。

### grule库的使用
#### KnowledgeLibrary
```go
lib := ast.NewKnowledgeLibrary()
```
knowledgeLibrary对象是众脚本的容器，其实质是一个map对象。
#### RuleBuilder
```go
ruleBuilder := builder.NewRuleBuilder(lib)
```
ruleBuilder 是脚本的编译构建工具,这里通过lib创建ruleBuilder，为了在编译脚本的时候避免重复编译。

#### build
```go
ruleBuilder.BuildRuleFromResource("TestFuncChaining", "0.0.1", pkg.NewBytesResource([]byte(rule)))
```
BuildRuleFromResource 这里使用字符rule来编译，形成KnowledgeBase指针对象存入lib中,前两个参数分别为Name和Version，用于唯一标识一个KnowledgeBase对象。

#### knowledgeBase
```go
kb := lib.NewKnowledgeBaseInstance("TestFuncChaining", "0.0.1")
```
指定Name和Version获取一个KnowledgeBase的对象，注意这里是获取一个拷贝对象，用来保证并发多次获取对象的时候，不会相互影响。

#### DataContext
```go
dataContext := ast.NewDataContext()
err := dataContext.Add("Fact", Fact)
```
前面脚本例子中的Fact就是通过dataContext注入脚本的，这样就能在脚本中引用该对象的成员和方法。
这里的Fact必须是指针对象才可以。

#### Execute
```go
eng1 := &engine.GruleEngine{MaxCycle: 1}
err = eng1.Execute(dataContext, kb)
```
为了执行脚本，需要创建一个GruleEngine对象，设置MaxCyle用来指定脚本执行的次数。
Execute方法接受DataContext参数和KnowledgeBase对象，使用dataContext表示的上下文执行KnowledgeBase的脚本逻辑，完成对象的判断和修改。

#### 完整的例子
```go
	dataContext := ast.NewDataContext()
	err := dataContext.Add("Fact", Fact)
	assert.NoError(t, err)

	lib := ast.NewKnowledgeLibrary()
	ruleBuilder := builder.NewRuleBuilder(lib)
	err = ruleBuilder.BuildRuleFromResource("TestFuncChaining", "0.0.1", pkg.NewBytesResource([]byte(rule)))
	kb := lib.NewKnowledgeBaseInstance("TestFuncChaining", "0.0.1")
	eng1 := &engine.GruleEngine{MaxCycle: 1}
	err = eng1.Execute(dataContext, kb)
```

## 进阶
### 多脚本判断
```
rule  example1  "example1-rule"  salience 1 {
when
(Fact.Distance > 5000  &&   Fact.Duration > 120) && (Fact.Result == false)
Then
   Fact.NetAmount=143.320007;
   Fact.SetResult(true);
}

rule  example2  "example2-rule"  salience 1 {
when
(Fact.Distance < 5000  ||  Fact.Duration < 120) && (Fact.Result == true)
Then
   Fact.NetAmount=143.320007;
   Fact.SetResult(true);
}
```
脚本中是可以含有多个rule的，这些rule会同时进行条件判断，然后根据权重高低来决定执行单元。

### MaxCycle
MaxCycle的含义为退出前的最大循环次数，因为脚本的执行一次只会选出一个rule进行执行，然后符合条件的rule可能会有多个，甚至一个rule在执行完之后条件依旧成立，所以MaxCycle会控制最大循环次数。

## 总结
grule-rule-engine 给予了你一个能力，让程序能根据输入的字符串改变变量的值，调用对应的方法。这样的方法就类似给静态编译的语言一个动态执行的能力。基于这样的能力，我们可以在数据链路中加入各种值，在rpc请求中传递元数据。

## 猜想
grule的实现是基于antlr的语法树解析，生成的接口代码位于antlr/parser/grulev3中，我们能在其中找到语法解析后各阶段的处理函数，例如EnterWhenScope,ExitWhenScope等。
引用对象的成员和调用对象的方法都是通过反射reflect来实现的，

## 参考资料
[grule-rule-engine](https://github.com/hyperjumptech/grule-rule-engine)


---

欢迎加入 GOLANG 中国社区：[https://gocn.vip](https://gocn.vip)