# Table

GenORMではSQL文を実行するメソッドチェーンの最初にテーブルを値として渡します。

## 通常のテーブル
Joinを含まない通常のテーブルを表す値は、生成コード中にある設定ファイルの構造体名と同名の関数を呼び出すことで得られます。
例えば、コード生成の例で使った、`users`テーブルであれば設定ファイルの構造体の名前は`User`であるため、生成コードの`User`関数を呼び出すことでテーブルに対応する値を得られます。つまり、コード生成時に指定した`package`フラッグの値が`orm`であれば、`orm.User()`が`users`テーブルに対応する値となります。

### 例
```go
// `users`テーブル
orm.User()

// `messages`テーブル
orm.Message()
```

## Join後のテーブル
Joinを行い作られるテーブル、例えば`users INNER JOIN messages ON users.id = messages.user_id`のようなテーブルは、Joinの元となるテーブル(`users`)を表す値(`orm.User()`)のJoinするテーブルに対応するメソッドを呼び出し、次にJoinの種類に対応するメソッド(`Join`)を呼び出すことで作成できます。
また、Joinの種類に対応するメソッドは引数としてON句の条件を取ります。

### 例
```go
// Join後のテーブルとJoin前のテーブルは異なるため、カラムの型変換が必要
userID := orm.MessageUserParseExpr(user.ID)
messageUserID := orm.MessageUserParseExpr(message.UserID)

// `users INNER JOIN messages ON users.id = messages.user_id`
innerJoinedTable := orm.User().
	Message().Join(genorm.Eq(userID, messageUserID))

// `users CROSS JOIN messages
crossJoinedTable := orm.User().
	Message().Join(nil)

// `users LEFT JOIN messages ON users.id = messages.user_id`
leftJoinedTable := orm.User().
	Message().LeftJoin(genorm.Eq(userID, messageUserID))

// `users RIGHT JOIN messages ON users.id = messages.user_id`
rightJoinedTable := orm.User().
	Message().RightJoin(genorm.Eq(userID, messageUserID))
```
