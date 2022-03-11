# GenORM

Generics導入後のGoのためのクエリビルダーです。

### 特徴

ジェネリクスを用いてSQLの表現に適切なGoの型を対応づけることで

* Goの型が異なる値をSQL内で`=`などでの比較やUPDATEでの値の更新に使用した際にコンパイルエラーとなる
* 使用できないテーブルのカラム名を使用した際にコンパイルエラーとなる

など、コンパイルの時点で従来のGo言語のORMやクエリビルダーでは防げなかった多くのSQLのミスを発見できます。

また、SQLの多くのCRUDの構文をサポートしています。

#### 例1

文字列のカラム`` `users`.`name` ``は`string`の値と比較することはできますが、`int`の値と比較するとコンパイルエラーとなります。

```
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

```
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
