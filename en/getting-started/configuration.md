# Configuration

Describe the table configuration in the Configuration file using the Go structure. Since the subsequent code generation will generate structures and functions with the same identifier, it is recommended to set a different build tag from the main body of the code as shown below.

```go
//go:build genorm
// +build genorm
```

### Table

Tables are represented by structs. The table name is returned by the `TableName` method, which takes a reciever of the struct's pointer.

#### Example

`users` table

```go
type User struct {
    // Information on columns and joinable tables
}

func (*User) TableName() string {
    return "users"
}
```

### Column

Columns are set as fields of struct. The column name is set as the value of the `genorm` tag.

* `bool`
* `int`, `int8`, `int16`, `int32`, `int64`
* `uint`, `uint8`, `uint16`, `uint32`, `uint64`
* `float32`, `float64`
* `string`
* `time.Time`
* a type whose type itself implements `sql.Scanner` in `database/sql` and whose pointer implements `driver.Valuer` in `database/sql/driver`.
  * ex)`uuid.UUID`([github.com/google/uuid](https://github.com/google/uuid))

#### Example

The `users` table has `id`, `name` and `created_at` columns.

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

Indicates that the tables are joinable. A field of type `genorm.Ref[T]` represents a relation. The type parameter of `genorm.Ref` is a struct that represents a join destination table.
Also, currently the same table cannot be used twice in a Join.
Therefore, it is not possible to paste `genorm.Ref` into a `User` structure within a `User` structure, as it makes no sense.

In addition, duplicate prevention of identifiers after Join is currently incomplete.
For this reason, the generated code will cause an error when there is a table represented by the `MessageUserJoined` struct with a Relation pasted between the `User` struct and the `Message` struct.
In such cases, please adjust the structure name, etc.

#### Example

The `users` table can join the `messages` table.

```go
import "github.com/mazrean/genorm"

type User struct {
    // Column Information
    Message genorm.Ref[Message]
}

func (*User) TableName() string {
    return "users"
}

type Message struct {
    // Column Information
}

func (*Message) TableName() string {
    return "messages"
}
```
