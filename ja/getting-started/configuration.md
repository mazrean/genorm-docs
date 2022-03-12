# Configuration

Goの構造体を用いて、Configurationファイルにテーブル構成について記述します。この後のコード生成で同一識別子の構造体や関数が生成されるため、以下のようにコード本体とは異なるbuild tagを設定することをお勧めします。

```go
//go:build genorm
// +build genorm
```

### Table

テーブルはstructで表現します。また、テーブル名はstructのpointerをreceiverにとる`TableName`メソッドで返します。

#### 例

`users`テーブル

```go
type User struct {
    // ColumnやJoin可能なテーブルの情報
}

func (*User) TableName() string {
    return "users"
}
```

### Column

カラムについてはstructのfieldとして設定します。カラム名は`genorm`タグの値として設定します。fieldの型には以下が使用できます。

* `bool`
* `int`, `int8`, `int16`, `int32`, `int64`
* `uint`, `uint8`, `uint16`, `uint32`, `uint64`
* `float32`, `float64`
* `string`
* `time.Time`
* その型自体が`database/sql`の`sql.Scanner`を実装しており、pointerが`database/sql/driver`の`driver.Valuer`を実装している型
  * ex)`uuid.UUID`([github.com/google/uuid](https://github.com/google/uuid))

#### 例

`users`テーブルに`id`、`name`、`created_at`カラムがある場合

```go
import (
    "time"
    "github.com/google/uuid"
)

type User struct {
	ID        uuid.UUID `genorm:"id"`
	Name      string    `genorm:"name"`
	CreatedAt time.Time `genorm:"created_at"`
}
```

### Relation

テーブルがJoin可能であることを示します。型が`genorm.Ref[T]`であるfieldでRelationを表します。`genorm.Ref`の型パラメーターにはJoin先のテーブルを表すstructを指定します。
また、現在同じテーブルを2回Joinで使うことができません。
このため、`User`構造体内で`User`構造体へ`genorm.Ref`を貼ることは意味がないためできません。

また、現在Join後の識別子の重複防止が不完全です。
このため、`User`構造体と`Message`構造体の間にRelationが貼られており、`MessageUserJoined`という構造体で表されるテーブルがある時などに生成されたコードがエラーとなります。
このような場合には構造体名などの調整をしてください。

#### 例

`users`テーブルに`messages`テーブルをJoinできる場合

```go
import "github.com/mazrean/genorm"

type User struct {
    // Columnの情報
    Message genorm.Ref[Message]
}

func (*User) TableName() string {
    return "users"
}

type Message struct {
    // Columnの情報
}

func (*Message) TableName() string {
    return "messages"
}
```
