# GenORM

SQL Builder to prevent SQL mistakes using the Golang generics

repository: https://github.com/mazrean/genorm

### Feature

By mapping SQL expressions to appropriate golang types using generics, you can discover many SQL mistakes at the time of compilation that are not prevented by traditional Golang ORMs or query builders.

For example:

* Compilation error occurs when using values of different Go types in SQL for = etc. comparisons or updating values in UPDATE statements
* Compile error occurs when using column names of unavailable tables

It also supports many CRUD syntaxes in SQL.

#### Example 1

String column `` `users`. `name` `` can be compared to a `string` value, but comparing it to an `int` value will result in a compile error.

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

#### Example 2

You can use an `id` column from the `users` table in a `SELECT` statement that retrieves data from the `users` table, but using an `id` column from the `messages` table will result in a compile error.

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

### Differences from other tools
Explain how this differs from existing GORM and ent.

#### GORM
GORM uses `interface{}` to flexibly accept arguments, allowing for intuitive query construction.
On the other hand, type constraints are very weak.
For this reason, there are few bugs that can be played at compile time, and bugs can easily be introduced.
Another problem is that it is difficult to understand the SQL that is actually executed, and it is easy for the SQL to behave differently from what is expected.

In contrast, GenORM uses type constraints to find as many bugs as possible at compile time.
We are also conscious of making it easy to imagine the SQL to be executed from the Go code, and we hope that the example code so far has been easy to imagine the SQL to be executed from the code.

Thus, GenORM and GORM differ greatly in both philosophy and function.

#### ent
We believe that the most significant difference in functionality between Go and ent is the Golang types that can be used.
In ent, only a finite number of types can be used, including types corresponding to primitive types such as int and bool, plus time.Time and UUID.
In contrast, GenORM allows [any type that satisfies the conditions of the `genorm.ExprType`interface](./usage/value-type.html), and then sets restrictions such as only types of the same type can be compared.
This eliminates the need for unnecessary type conversions and allows for [stronger constraints to be set by using Defined Type](./advanced-usage/defined-type.html).

For example, by defining `UserID` and `MessageID` as unique types as shown below, constraints such as "user IDs and message IDs cannot be compared" can be specified.
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

This prevents mistakes, as code such as the following will result in a compilation error.
```go
// SELECT * FROM `messages` WHERE `messages`.`id`=`messages`.`user_id`
messageValues, err := genorm.
    Select(orm.Message()).
    Where(genorm.Eq(message.IDExpr, message.UserIDExpr)).
    GetAll(db)
```

It is also important to note that GenORM allows queries to be constructed with a method chain similar to SQL.
Because ent is an "entity framework," database operations are abstracted as entity operations.
This has its advantages, but as with GORM, it also has the aspect of making the SQL to be executed difficult to understand.
This increases the probability of unintentionally writing a process that has performance problems.
In this respect, GenORM makes it easy to understand the SQL to be executed, so such problems are less likely to occur.

### Mechanism
This section explains how the code example works.
The definition of `genorm.EqLit` is as follows.
```go
func EqLit[T Table, S ExprType](
	expr TypedTableExpr[T, S],
	literal S,
) TypedTableExpr[T, WrappedPrimitive[bool]] {
	// 省略
}
```
Notice the `TypedTableExpr[T, S]` part.
As you can see from the `[T Table, S ExprType]` part, `T` is the type that represents the table used by the SQL expression, and `S` is the Go language type corresponding to the SQL expression.
Thus, GenORM limits the columns that can be used and the values that can be compared by having the table used by the SQL expression and the corresponding type of the Go language as type parameters.

Thus, because SQL expressions are typed, SQL can be restricted to operators such as `>`, `<`, and `AND`, functions such as `COUNT()`, and in the future, database-specific functions as well.
