# Update

SQLのUPDATE文は`genorm.Update`関数を用いて実行できます。

`genorm.Update`関数で始まり、`Do`、`DoCtx`のいずれかで終わる必要がありますが、それ以外の順番は入れ替えても問題ありません。ただし、同一のメソッドチェーンないで2度`OrderBy`以外の同じメソッドを実行した場合、実行時に`Do`、`DoCtx`がクエリを実行せずにエラーを返します。

#### 例

```
// UPDATE `users` SET `name`="name"
affectedRows, err = genorm.
    Update(orm.User()).
    Set(
        genorm.AssignLit(user.Name, genorm.Wrap("name")),
    ).
    Do(db)

// UPDATE `users` SET `name`="name"
affectedRows, err = genorm.
    Update(orm.User()).
    Set(
        genorm.AssignLit(user.Name, genorm.Wrap("name")),
    ).
    DoCtx(context.Background(), db)

// UPDATE `users` SET `name`="name" WHERE `id`={{uuid.New()}} 
affectedRows, err = genorm.
    Update(orm.User()).
    Set(
        genorm.AssignLit(user.Name, genorm.Wrap("name")),
    ).
    Where(genorm.EqLit(user.ID, uuid.New())).
    Do(db)

// UPDATE `users` SET `name`="name" ORDER BY `created_at` LIMIT 1
affectedRows, err = genorm.
    Update(orm.User()).
    Set(
        genorm.AssignLit(user.Name, genorm.Wrap("name")),
    ).
    OrderBy(genorm.Desc, user.CreatedAt).
    Limit(1).
    Do(db)
```

### Update

`UPDATE`文を発行するメソッドチェーンを開始する関数です。第1引数にCLIにより生成されたテーブルを受け取ります。

### Set(required)

`UPDATE`で値を更新するカラムを値の組みを設定します。引数には可変長の`*TableAssignExpr[T]`型を受け取ります。`*TableAssignExpr[T]`型の値は`genorm.Assign`関数または`genorm.AssignLit`関数の返り値として得られます。

### Where(optional)

`UPDATE`文に`WHERE`句を設定します。第1引数にGo言語での`bool`に対応するexpressionを受け取ります。

### Do

クエリを実行し、メソッドチェーンを終了します。引数には`database/sql`の`*sql.DB`や`*sql.Tx`を含む`genorm.DB`interfaceを満たす値を受け取り、これを使用してクエリを実行します。

### DoCtx

クエリを実行し、メソッドチェーンを終了します。第二引数には`database/sql`の`*sql.DB`や`*sql.Tx`を含む`genorm.DB`interfaceを満たす値を受け取り、これを使用してクエリを実行します。また、第一引数で`context.Context`を受け取ります。
