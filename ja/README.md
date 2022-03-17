# GenORM

Go言語のジェネリクスを利用したSQLのミスを防ぐSQL Builder

リポジトリ: https://github.com/mazrean/genorm

### 特徴

ジェネリクスを用いてSQLの表現に適切なGoの型を対応づけることで

* Goの型が異なる値をSQL内で`=`などでの比較やUPDATEでの値の更新に使用した際、コンパイルエラーとなる
* 使用できないテーブルのカラム名を使用した際にコンパイルエラーとなる

など、コンパイルの時点で従来のGo言語のORMやクエリビルダーでは防げなかった多くのSQLのミスを発見できます。

また、SQLの多くのCRUDの構文をサポートしています。

#### 例1

文字列のカラム`` `users`.`name` ``は`string`の値と比較することはできますが、`int`の値と比較するとコンパイルエラーとなります。

```go
// correct
userValues, err := genorm.
	Select(orm.User()).
	Where(genorm.EqLit(user.NameExpr, genorm.Wrap("name"))).
	GetAll(db)

// compile error
userValues, err := genorm.
	Select(orm.User()).
	Where(genorm.EqLit(user.NameExpr, genorm.Wrap(1))).
	GetAll(db)
```

#### 例2

`users`テーブルに対する`SELECT`文で`users`テーブルの`id`カラムは使用できますが、`messages`テーブルの`id`カラムを使用するとコンパイルエラーとなります。

```go
// correct
userValues, err := genorm.
	Select(orm.User()).
	Where(genorm.EqLit(user.IDExpr, uuid.New())).
	GetAll(db)

// compile error
userValues, err := genorm.
	Select(orm.User()).
	Where(genorm.EqLit(message.IDExpr, uuid.New())).
	GetAll(db)
```

### 既存のツールとの違い
既存のGORMやentとの違いについて説明します。

#### GORM
GORMでは`interface{}`を利用して柔軟に引数を受け入れることで、直感的にクエリを組み上げられるようになっています。
反面、型による制約が非常に弱くなっています。
このため、コンパイル時に弾けるバグが少なく、バグが混入しやすくなります。
また、実際に実行されるSQLがわかりづらく、想定したのと異なる挙動をしやすい、という問題もあるように思います。

対して、GenORMは型による制約でコンパイル時にできる限り多くのバグを見つけることができます。
また、Goのコードから実行されるSQLがイメージしやすいようにすることも意識しており、ここまでの例のコードでもコードから実行するSQLがイメージしやすかったのではないかと思います。

このように、GenORMとGORMでは思想、機能ともに大きく異なります。

#### ent
entとの機能面での最も大きな違いは使用できるGo言語の型と考えています。
entではint、boolなどのプリミティブ型に対応する型にtime.TimeやUUIDなどを加えた、有限個の型のみが使用できます。
対して、GenORMでは`genorm.ExprType`interfaceの条件を満たす任意の型[^2]を使用し、その上で同一の型でのみ比較可能、などの制約が設定されています。
これにより、不要な型変換を行う必要がなく、また、Defined Typeを用いることでより強力な制約を設定できるようになっています[^3]。

例えば、以下のように`UserID`と`MessageID`を独自型として定義することで、「ユーザーのIDとメッセージのIDは比較できない」といった制約が指定できます。
```go
type MessageID uuid.UUID

func (mid *MessageID) Scan(src any) error {
	return (*uuid.UUID)(mid).Scan(src)
}

func (mid MessageID) Value() (driver.Value, error) {
	return uuid.UUID(mid).Value()
}

type UserID uuid.UUID

func (uid *UserID) Scan(src any) error {
	return (*uuid.UUID)(uid).Scan(src)
}

func (uid UserID) Value() (driver.Value, error) {
	return uuid.UUID(uid).Value()
}
```

これにより、以下のようなコードはコンパイルエラーとなりミスを防げます。
```go
// SELECT * FROM `messages` WHERE `messages`.`id`=`messages`.`user_id`
messageValues, err := genorm.
    Select(orm.Message()).
    Where(genorm.Eq(message.IDExpr, message.UserIDExpr)).
    GetAll(db)
```

また、GenORMではSQLに近いメソッドチェーンでクエリを構築できる点も重要です。
entは「entity framework」であるため、データベースの操作がentityの操作として抽象化されています。
これはメリットもありますが、GORMと同様に実行されるSQLがわかりづらくなるという側面もあります。
これは、意図せずパフォーマンスに問題のある処理を書いてしまう確率を上昇させてしまいます。
この点、GenORMでは実行されるSQLがわかりやすいため、このような問題は起こりづらいでしょう。

[^2]: 詳細は https://mazrean.github.io/genorm-docs/ja/usage/value-type.html 参照
[^3]: 詳細は https://mazrean.github.io/genorm-docs/ja/advanced-usage/defined-type.html

### 仕組み
コード例の動作の仕組みを解説します。
`genorm.EqLit`の定義は
```go
func EqLit[T Table, S ExprType](
	expr TypedTableExpr[T, S],
	literal S,
) TypedTableExpr[T, WrappedPrimitive[bool]] {
	// 省略
}
```
のようになっています。
注目してほしいのが`TypedTableExpr[T, S]`の部分です。
`[T Table, S ExprType]`の部分からもわかるように、`T`はSQLのexpressionが使用しているテーブルを表す型、`S`はSQLのexpressionに対応するGo言語の型となっています。
このように、GenORMではSQLのexpressionに使用しているテーブルとGo言語の対応する型を型パラメーターとして持たせることで、使用可能なカラムや比較して良い値に制限をかけています。

このように、SQLのexpressionに型をつけているため、`>`や`<`、`AND`などの演算子、`COUNT()`などの関数、さらに将来的にはデータベース独自の関数にも制限をかけてSQLを実行できます。
