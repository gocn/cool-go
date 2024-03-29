# sql发布管理工具 gopg-migrations

## 1. gopg-migrations 是什么？
[gopg-migrations](github.com/go-pg/migrations/v7) 在我们使用数据库时，每次发布都要改一些表结构，或者修改数据之类的操作。

这些操作，手动执行既危险，又容易忘记；当新人看到奇怪的表结构，感觉到很诧异。如果我们能将所有操作管理起来，本地测试好之后再发布上去，那么就不会有上述这些问题。

因此，gopg-migrations这个库，给我们带来非常大的便利。当然，这里是对postgres数据库的支持。

## 2. 怎么使用

下载扩展库 `github.com/go-pg/migrations/v7` 即可使用。

2.1 假如我们先创建一个数据库 students，然后调用 migrations 去执行代码。

   步骤如下：
- 编写创建表的sql，使用migrations目录管理起来，num_xxx 表示按照num顺序执行
- 主逻辑
   
main.go 主逻辑如下：
```go
package main

import (
	"fmt"
	"time"

	pg "github.com/test_go_pg/pg"

	"go.uber.org/zap"
)

func main() {
	fmt.Println("Starting go-pg-migrations...")

	// Bootstrap check pg
	if err := pg.PGDBWrite.Ping(); err != nil {
		fmt.Println(pg.DBWriteConnectionError, zap.Error(err))
		return
	}
	fmt.Println("PostgreSQL is running",
		zap.String("user", pg.PGDBWrite.Options().User),
		zap.String("addr", pg.PGDBWrite.Options().Addr),
		zap.String("db", pg.PGDBWrite.Options().Database))

	// Migrate to latest pg schema
	if err := pg.PGDBWrite.Migrate(); err != nil {
		fmt.Println(pg.DBMigrationError, zap.Error(err))
		return
	}

	time.Sleep(time.Hour)
	fmt.Println("go-pg-migrations is stopping...")
}

```

gopg-migrations相关函数编码如下：
```go
package pg

import (
	"context"
	"fmt"
	"time"

	"github.com/go-pg/migrations/v7"
	"github.com/go-pg/pg/v9"
	"go.uber.org/zap"
)

type TGPGDB struct {
	*pg.DB
}

func NewTGPGDB(db *pg.DB) *TGPGDB {
	return &TGPGDB{db}
}

type TGPGDBOptions struct {
	Url string

	DisableBeforeQueryLog bool
	DisableAfterQueryLog  bool
}

func CreateTGPGDB(url string) *TGPGDB {
	return CreateTGPGDBWithOptions(&TGPGDBOptions{Url: url})
}

func CreateTGPGDBWithOptions(dbOpts *TGPGDBOptions) *TGPGDB {
	opts, err := pg.ParseURL(dbOpts.Url)
	if err != nil {
		fmt.Println(DBURLParseError, zap.String("URL", dbOpts.Url), zap.Error(err))
		return nil
	}
	opts.ReadTimeout = DBReadTimeout
	opts.WriteTimeout = DBWriteTimeout
	opts.TLSConfig = nil // disabled for faster local connection (even in production)
	if DBNumConns > 0 {
		opts.PoolSize = DBNumConns
	}

	db := NewTGPGDB(pg.Connect(opts))
	return db
}

// Ping simulates a "blank query" behavior similar to lib/pq's
// to check if the db connection is alive.
func (db *TGPGDB) Ping() error {
	_, err := db.ExecOne("SELECT 1")
	return err
}

// Migrate check and migrate to lastest db version.
func (db *TGPGDB) Migrate() error {
	// Make sure to only search specified migrations dir
	cl := migrations.NewCollection()
	cl.DisableSQLAutodiscover(true)
	err := cl.DiscoverSQLMigrations(DBMigrationsDir)
	if err != nil {
		return err
	}

	var oldVersion, newVersion int64
	// Run all migrations in a transaction so we rollback if migrations fail anywhere
	err = db.RunInTransaction(func(tx *pg.Tx) error {
		// Intentionally ignore harmless errors on initializing gopg_migrations
		_, _, err = cl.Run(db, "init")
		//if err != nil && !DBMigrationsAlreadyInit(err) {
		if err != nil{
			return err
		}
		oldVersion, newVersion, err = cl.Run(db, "up")
		return err
	})
	if err != nil {
		return err
	}
	if newVersion == oldVersion {
		fmt.Println("db schema up to date")
	} else {
		fmt.Println("db schema migrated successfully", zap.Int64("from", oldVersion), zap.Int64("to", newVersion))
	}
	return nil
}

// WithContextTimeout
func WithContextTimeout(ctx context.Context, f func(ctx context.Context)) {
	WithContextTimeoutValue(ctx, DBStmtTimeout, f)
}

// WithContextTimeoutValue
func WithContextTimeoutValue(ctx context.Context, timeout time.Duration, f func(ctx context.Context)) {
	// check context timeout setting with upper bound read/write limit
	if timeout > DBReadTimeout && timeout > DBWriteTimeout {
		fmt.Println(DBContextTimeoutExceedUpperBound,
			zap.Error(fmt.Errorf("query timeout %s exceed upper bound (%s|%s)", timeout, DBReadTimeout, DBWriteTimeout)))
	}
	newCtx, cancel := context.WithTimeout(ctx, timeout)
	f(newCtx)
	cancel()
}

```


直接运行 go run main.go，此时代码运行，会生成students数据表：

sql如下：
![alt 条形图](https://swarm-gateways.net/bzz:/5aac64589b094632861794b07fc658ac9bef554d2956863798666f791dfa6f19/pg1.png)


执行后，会创建号数据表： students 和 gopg-migrations。
![alt 条形图](https://swarm-gateways.net/bzz:/a17cb08722a105678eb03210ebb785e7131e5d21316c69e3600cd47aeb2479e5/pg6.png)

查看 gopg-migrations如下：

![alt 条形图](https://swarm-gateways.net/bzz:/50e130d4a8337e545536cf249b435c83b527c38f0c3003808e79a7c7704e3bcf/pg2.png)



2.2 假设我们要增加sql语句，怎么做呢？

- 直接添加sql
- 重新运行
   
sql如下：
![alt 条形图](https://swarm-gateways.net/bzz:/290f87a7d93dd43651b5f3842d8059e763237551b5ca2fa44135873a7be25ecb/pg3.png)

执行如下：
![alt 条形图](https://swarm-gateways.net/bzz:/84ee116ba02b6480f63c9538571f881a98dca0b0cd7f40c3fdf0b64cb8963abb/pg4.png)

查看students数据表，发现1，2操作都已经执行了：
![alt 条形图](https://swarm-gateways.net/bzz:/3754e6aea9b6e359d4ed7dcb4360f16bd23e1abf01d5e29859b30c76abe0a6c4/pg5.png)

2.3.管理sql语句
类似1，2中的方式，我们可以添加更多的sql语句，把sql管理起来。



## 总结

[gopg-migrations](github.com/go-pg/migrations/v7) 这个库，能够把数据库所有的操作sql全部管理起来，一方面可以让我们清晰的看到sql项目的发展历程，另一方面本地测试后发布更加安全！


以上所有内容均采用最新官方案例做示例
## 参考资料

- [查看全部代码](https://github.com/turingczz/test_go_pg)
- [图片上传及查看，使用swarm网关实现](https://swarm-gateways.net/)
