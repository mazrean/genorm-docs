# Value Type

GenORMではSQLの表現を以下の3つの種類に分類し、各SQLの表現にGoの型を対応させることでSQLのミスを防止しています。

* Literal
* Expr
* Column

また、`uuid.UUID`などの「その型自体が[sql.Scanner](https://pkg.go.dev/database/sql#Scanner)を実装しており、pointerが[driver.Valuer](https://pkg.go.dev/database/sql/driver#Valuer)を実装している型」を`genorm.ExprType`interfaceに入れることができます。

### Literal

SQLでLiteralとして扱われる値です。

`genorm.ExprType`interfaceに当てはまる型の値がLiteralにあたります。

`bool`、`int`、`float32`などの`genorm.ExprType`interfaceに入れられないプリミティブ型の値は`genorm.Wrap`関数を用いて`genorm.WrappedPrimitive[T]`型とすることで、Literalとして扱えます。

### Expr

SQLでexpressionとして扱う値です。

この値はExprを構成する中で使用したテーブルの型とExprが対応する`genorm.ExprType`interfaceに当てはまる型を型パラメーターとして持ちます。

例えば、`genorm.EqLit(user.IDExpr, uuid.New())`の返り値はusersテーブルのidカラムを使用しており、SQLの中で真偽値となるため、`*orm.UserTable`と`bool`を型パラメーターとして持つExprとなります。

テーブルのカラムは`users`テーブルの`id`カラムであれば`users.IDExpr`のように、テーブルに対応するpackage内の`~Expr`という名前の変数を用いることで`Expr`として使用できます。

### Column

SQLでカラム名として扱われる値です。

この値はカラムを含むテーブルに対応する型とExprが対応する`genorm.ExprType`interfaceに当てはまる型を型パラメーターとして持ちます。
