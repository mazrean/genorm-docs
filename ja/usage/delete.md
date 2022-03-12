# Delete

SQLの`DELETE`文は`genorm.Delete`関数を用いて実行できます。

`genorm.Delete`関数で始まり、`Do`、`DoCtx`のいずれかで終わる必要がありますが、それ以外の順番は入れ替えても問題ありません。ただし、同一のメソッドチェーン内で2度`OrderBy`以外の同じメソッドを実行した場合、実行時に`Do`、`DoCtx`がクエリを実行せずにエラーを返します。

#### 例

```go
// DELETE FROM `users`
affectedRows, err = genorm.
    Delete(orm.User()).
    Do(db)

// DELETE FROM `users`
affectedRows, err = genorm.
    Delete(orm.User()).
    DoCtx(context.Background(), db)

// DELETE FROM `users` WHERE `id`={{uuid.New()}} 
affectedRows, err = genorm.
    Delete(orm.User()).
    Where(genorm.EqLit(user.IDExpr, uuid.New())).
    Do(db)

// DELETE FROM `users` ORDER BY `created_at` LIMIT 1
affectedRows, err = genorm.
    Delete(orm.User()).
    OrderBy(genorm.Desc, user.CreatedAt).
    Limit(1).
    Do(db)
```

### Delete

`DELETE`文を発行するメソッドチェーンを開始する関数です。第1引数にCLIにより生成されたテーブルを受け取ります。`DELETE`文はJoinされたテーブルに対しては実行できないため、Joinされたテーブルは与えられません。

### Where(optional)

`DELETE`文に`WHERE`句を設定します。第1引数にGo言語での`bool`に対応するexpressionを受け取ります。

### Do

クエリを実行し、メソッドチェーンを終了します。引数には[`*sql.DB`](https://pkg.go.dev/database/sql#DB)/[`*sql.Tx`](https://pkg.go.dev/database/sql#Tx)を含む`genorm.DB`interfaceを満たす値を受け取り、これを使用してクエリを実行します。

### DoCtx

クエリを実行し、メソッドチェーンを終了します。第二引数には[`*sql.DB`](https://pkg.go.dev/database/sql#DB)/[`*sql.Tx`](https://pkg.go.dev/database/sql#Tx)を含む`genorm.DB`interfaceを満たす値を受け取り、これを使用してクエリを実行します。また、第一引数で`context.Context`を受け取ります。contextがキャンセルされるとデータベースとのコネクションが解放され、無駄なコネクションの使用を防げます。
