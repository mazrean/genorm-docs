# Insert

SQLの`INSERT`文は`genorm.Insert`関数を用いて実行できます。

`genorm.Insert`関数で始まり、`Do`または`DoCtx`で終わる必要がありますが、それ以外の順番は入れ替えても問題ありません。ただし、同一のメソッドチェーンないで2度`Values`や`Field`を実行した場合、実行時に`Do`や`DoCtx`がクエリを実行せずにエラーを返します。

#### 例

```
// INSERT INTO `users` (`id`, `name`, `created_at`) VALUES ({{uuid.New()}}, "name", {{time.Now()}})
affectedRows, err := genorm.
    Insert(orm.User()).
    Values(&orm.UserTable{
        ID: uuid.New(),
        Name: genorm.Wrap("name"),
        CreatedAt: time.Now(),
    }).
    Do(db)

// INSERT INTO `users` (`id`, `name`) VALUES ({{uuid.New()}}, "name")
affectedRows, err := genorm.
    Insert(orm.User()).
    Fields(user.ID, user.Name).
    Values(&orm.UserTable{
        ID: uuid.New(),
        Name: genorm.Wrap("name"),
    }).
    Do(db)

// INSERT INTO `users` (`id`, `name`, `created_at`) VALUES ({{uuid.New()}}, "name", {{time.Now()}})
affectedRows, err := genorm.
    Insert(orm.User()).
    Values(&orm.UserTable{
        ID: uuid.New(),
        Name: genorm.Wrap("name"),
        CreatedAt: time.Now(),
    }).
    DoCtx(context.Background(), db)
```

### Insert

INSERT文を発行するメソッドチェーンを開始する関数です。第一引数にCLIにより生成されたテーブルを受け取ります。INSERT文はJoinされたテーブルに対しては実行できないため、Joinされたテーブルは与えられません。

### Fields(optional)

`INSERT`の対象のカラムを指定します。デフォルトではテーブルの全てのカラムが`INSERT`の対象となります。

### Values(required)

INSERTする値を指定します。メソッドチェーンがDoやDoCtxで終了する前にちょうど1回呼び出す必要があります。FieldsでINSERTの対象となっていないカラムに値が入っていた場合は無視されます。

### Do

クエリを実行し、メソッドチェーンを終了します。引数には`database/sql`の`*sql.DB`や`*sql.Tx`を含む`genorm.DB`interfaceを満たす値を受け取り、これを使用してクエリを実行します。

### DoCtx

クエリを実行し、メソッドチェーンを終了します。第二引数には`database/sql`の`*sql.DB`や`*sql.Tx`を含む`genorm.DB`interfaceを満たす値を受け取り、これを使用してクエリを実行します。また、第一引数で`context.Context`を受け取ります。