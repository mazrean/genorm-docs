# Connecting to Database

GenORM executes queries using [`*sql.DB`](https://pkg.go.dev/database/sql#DB)/[`*sql.Tx`](https://pkg.go.dev/database/sql#Tx). Therefore, connection to the database can be made in the same way as in `database/sql`. However, we have only confirmed that it works with MySQL and MariaDB.

#### Example

```go
import (
  "database/sql"
  _ "github.com/go-sql-driver/mysql"
)

db, err := sql.Open("mysql", "user:pass@tcp(host:port)/database?parseTime=true&loc=Asia%2FTokyo&charset=utf8mb4")
```
