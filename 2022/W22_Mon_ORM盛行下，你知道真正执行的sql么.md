# ORM 盛行下，你知道真正执行的 sql 么

## 推荐理由

在这个 orm 盛行的服务端开发环境下，已经很少有人写 raw sql 了吧，毕竟 orm 框架可以帮助开发人员屏蔽 db 的细节，同时还能在一定成程度上预防 sql 注入的风险，大多数情况下，对业务代码的单测，会使用 sqlite 来 mock 真正的 db，以验证功能的完备性，但当你写完一条 orm 语句后，是否会校验，生成的真正执行的 sql 是你的预期么？是否会校验，在代码变更的时候 sql 语句是否也发生了变更呢？针对以上两个问题，sqlmock 可以完成对 sql 语句的单侧，让你对 orm 生成的 sql 了如指掌，同时清晰 test raw sql 也让 review 的同事快乐加倍。

## 实现原理

其实 sql mock 只是实现了 go  sql/driver 的接口，用于模拟数据库的连接，本质上并不会进行存储数据，只能特定的返回。

## 实际栗子

> 通过 sqlmock 测试创建 user 的sql语句
> 

```go
package main

import (
	"github.com/DATA-DOG/go-sqlmock"
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
	"testing"
)

type User struct {
	ID int64 `gorm:"column:id"`
	UserName string `gorm:"column:user_name"`
}

func (User) TableName() string {
	return "users"
}

func CreateUser(db *gorm.DB, user *User) error{
	return db.Table(user.TableName()).Create(&user).Error
}

const (
	rawSQLCreateUser = "INSERT INTO `users` (`user_name`,`id`) VALUES (?,?)"
)

func TestShouldUpdateStats(t *testing.T) {
	db, mock, err := sqlmock.New(sqlmock.QueryMatcherOption(sqlmock.QueryMatcherEqual))
	if err != nil {
		t.Fatalf("an error '%s' was not expected when opening a stub database connection", err)
	}
	defer db.Close()
	dialector := mysql.New(mysql.Config{
		DriverName:                "mysql",
		Conn:                      db,
		SkipInitializeWithVersion:true,
	})
	user := &User{
		ID:       1,
		UserName: "zr",
	}
	gdb, err := gorm.Open(dialector, &gorm.Config{})
	gdb = gdb.Debug()
	mock.ExpectBegin()
	mock.ExpectExec(rawSQLCreateUser).
		WithArgs(user.UserName,user.ID).WillReturnResult(sqlmock.NewResult(1, 1))
	mock.ExpectCommit()
	if err = CreateUser(gdb, user); err != nil {
		t.Errorf("gorm create fail, err=%v", err)
	}
	if err = mock.ExpectationsWereMet(); err != nil {
		t.Errorf("unexcept sql, err=%v", err)
	}
}
```

## 总结

> PS： 笔者的老板对 sql 治理比较看重，所以在业务实现中会在存储层对于每一个 sql 写 sqlmock。
> 

sqlmock 虽然不能对整个业务功能进行集成测试，但是在 dao 层，对存储数据的操作进行 sql 语句层面的校验个人觉得是有必要的，一方面这会使 sql 的管理更加高效（将定义的 rawsql 单独放在一个文件中），另一方面，数据库层面改动时，有完整的 sql 校验逻辑，方便 review 的人直观的看出来具体数据有什么变化。

顺带一提：这个库的开源作者想要找个开源维护者，感兴趣的也可以参与尝试下。

## 参考

github: [https://github.com/DATA-DOG/go-sqlmock](https://github.com/DATA-DOG/go-sqlmock)