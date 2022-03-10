# Select

SQLの`SELECT`文は`genorm.Select`関数または`genorm.Pluck`関数を用いて実行できます。

`genorm.Select`関数で始まり、`Get`、`GetCtx`、`GetAll`、`GetAllCtx`のいずれかで終わる必要がありますが、それ以外の順番は入れ替えても問題ありません。ただし、同一のメソッドチェーンないで2度`OrderBy`以外の同じメソッドを実行した場合、実行時に`Get`、`GetCtx`、`GetAll`、`GetAllCtx`がクエリを実行せずにエラーを返します。

#### 例

```
// SELECT `id`, `name`, `created_at` FROM `users`
// userValues: []orm.UserTable
userValues, err := genorm.
	Select(orm.User()).
	GetAll(db)

// SELECT `id`, `name`, `created_at` FROM `users`
// userValues: []orm.UserTable
userValues, err := genorm.
	Select(orm.User()).
	GetAllCtx(context.Background(), db)

// SELECT `id`, `name`, `created_at` FROM `users` WHERE `id` = {{uuid.New()}}
// userValues: []orm.UserTable
userValues, err := genorm.
	Select(orm.User()).
	Where(genorm.EqLit(user.IDExpr, uuid.New())).
	GetAll(db)

// SELECT `name`, `created_at` FROM `users`
// userValues: []orm.UserTable
userValue, err := genorm.
	Select(orm.User()).
	Fields(user.Name, user.CreatedAt).
	GetAll(db)

// SELECT `id`, `name`, `created_at` FROM `users` LIMIT 1
// userValue: orm.UserTable
userValue, err := genorm.
	Select(orm.User()).
	Get(db)

// SELECT `id`, `name`, `created_at` FROM `users` LIMIT 1
// userValue: orm.UserTable
userValue, err := genorm.
	Select(orm.User()).
	GetCtx(context.Background(), db)

// SELECT `id`, `name`, `created_at` FROM `users` ORDER BY `created_at` DESC
// userValues: []orm.UserTable
userValues, err := genorm.
	Select(orm.User()).
	OrderBy(genorm.Desc, user.CreatedAt).
	GetAll(db)

// SELECT `id`, `name`, `created_at` FROM `users` ORDER BY `created_at` DESC, `id` ASC
// userValues: []orm.UserTable
userValues, err := genorm.
	Select(orm.User()).
	OrderBy(genorm.Desc, user.CreatedAt).
	OrderBy(genorm.Asc, user.ID).
	GetAll(db)

// SELECT DICTINCT `id`, `name`, `created_at` FROM `users`
// userValues: []orm.UserTable
userValues, err := genorm.
	Select(orm.User()).
	Distinct().
	GetAll(db)

// SELECT `id`, `name`, `created_at` FROM `users` LIMIT 5 OFFSET 3
// userValues: []orm.UserTable
userValues, err := genorm.
	Select(orm.User()).
	Limit(5).
	Offset(3).
	GetAll(db)

// SELECT `name` FROM `users` GROUP BY `name` HAVING COUNT(`id`) > 10
// userValues: []orm.UserTable
userValues, err := genorm.
	Select(orm.User()).
	Fields(user.Name).
	GroupBy(user.Name).
	Having(genorm.Gt(genorm.Count(user.IDExpr), genorm.Wrap(int64(10)))).
	GetAll(db)

// SELECT `id`, `name`, `created_at` FROM `users` FOR UPDATE
// userValues: []orm.UserTable
userValues, err := genorm.
	Select(orm.User()).
	Lock(genorm.ForUpdate).
	GetAll(db)

// SELECT `id` FROM `users`
// userIDs: []uuid.UUID
userIDs, err := genorm.
	Pluck(orm.User(), user.IDExpr).
	GetAll(db)

// SELECT `id` FROM `users` LIMIT 1
// userID: uuid.UUID
userID, err := genorm.
	Pluck(orm.User(), user.IDExpr).
	Get(db)

// SELECT `users`.`name`, `messages`.`content` FROM `users` INNER JOIN `messages` ON `users`.`id` = `messages`.`user_id`
// messageUserValues: []orm.MessageUserTable
userID := orm.MessageUserParseExpr(user.ID)
userName := orm.MessageUserParseExpr(user.Name)
messageUserID := orm.MessageUserParseExpr(message.UserID)
messageContent := orm.MessageUserParseExpr(message.Content)
messageUserValues, err := genorm.
	Select(orm.User().
		Message().Join(genorm.Eq(userID, messageUserID))).
	Fields(userName, messageContent).
	GetAll(db)
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
