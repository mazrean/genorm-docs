# Configuration

Goの構造体を用いて、Configurationファイルにテーブル構成について記述します。この後のコード生成で同一識別子の構造体や関数が生成されるため、以下のようにコード本体とは異なるbuild tagを設定することをお勧めします。

```
//go:build genorm
// +build genorm
```

### Table

テーブルはstructで表現します。また、テーブル名はstructのpointerをrecieverにとる`TableName`メソッドで返します。

#### 例

`users`テーブル

```
type User struct {
    // ColumnやJoin可能なテーブルの情報
}

func (*User) TableName() string {
    return "users"
}
```

### Column

カラムについてはstructのfieldとして設定します。カラム名は`genorm`タグの値として設定します。fieldの型には

* `bool`
* `int`, `int8`, `int16`, `int32`, `int64`
* `uint`, `uint8`, `uint16`, `uint32`, `uint64`
* `float32`, `float64`
* `string`
* `time.Time`
* その型自体が`database/sql`の`sql.Scanner`を実装しており、pointerが`database/sql/driver`の`driver.Valuer`を実装している型
  * ex)`uuid.UUID`([github.com/google/uuid](https://github.com/google/uuid))

が使用できます。

#### 例

`users`テーブルに`id`、`name`、`created_at`カラムがある場合

```
import (
    "time"
    "github.com/google/uuid"
)

type User struct {
    ID uuid.UUID `genorm:"id"`
    Name string `genorm:"name"`
    CreatedAt time.Time `genorm:"created_at"`
}
```

### Relation

テーブルがJoin可能であることを示します。型が`genorm.Ref[T]`であるfieldでRelationを表します。`genorm.Ref`の型パラメーターにはJoin先のテーブルを表すstructを指定します。

#### 例

`users`テーブルに`messages`テーブルをJoinできる場合

```
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
