# Conectiong to Database

GenORMは[`*sql.DB`](https://pkg.go.dev/database/sql#DB)/[`*sql.Tx`](https://pkg.go.dev/database/sql#Tx)を用いてクエリを実行します。このため、データベースへの接続は`database/sql`と同様に行えます。ただし、現在動作確認はMySQL、MariaDBでしか行えていません。

#### 例

```go
import (
  "database/sql"
  _ "github.com/go-sql-driver/mysql"
)

db, err := sql.Open("mysql", "user:pass@tcp(host:port)/database?parseTime=true&loc=Asia%2FTokyo&charset=utf8mb4")
```
