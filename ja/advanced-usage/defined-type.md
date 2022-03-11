# Defined Type

`genorm.ExprType`interfaceを満たす独自型を定義することで、さらに強力にSQLのミスを防止できます。

Usageの例として使用したConfigurationファイルでは`messages`テーブルの`id`カラムと`user_id`カラムは共に`uuid.UUID`型となっています。このため、

```go
messageValues, err := genorm.
	Select(orm.Message()).
	Where(genorm.Eq(message.IDExpr, message.UserIDExpr)).
	GetAll(db)
```

のようなコードはコンパイルエラーとなりません。

しかし、messageのidとuserのidを比較したい場合はほとんどないはずです。

このような時に

```go
type MessageID uuid.UUID

func (mid MessageID) Scan(src any) error {
    return uuid.UUID(mid).Scan(src)
}

func (mid *MessageID) Value() (driver.Value, error) {
    return (*uuid.UUID)(mid).Value()
}

type UserID uuid.UUID

func (uid UserID) Scan(src any) error {
    return uuid.UUID(uid).Scan(src)
}

func (uid *UserID) Value() (driver.Value, error) {
    return (*uuid.UUID)(uid).Value()
}
```

のようなDefined Typeを定義し、Configurationファイルの`messages`テーブルの`id`カラムと`user_id`カラムをそれぞれ`MessageID`型、`UserID`型にします。

このようにすることで、messageのidとuserのidを別の型として扱うことができ、

```go
messageValues, err := genorm.
	Select(orm.Message()).
	Where(genorm.Eq(message.IDExpr, message.UserIDExpr)).
	GetAll(db)
```

がコンパイルエラーとなります。
