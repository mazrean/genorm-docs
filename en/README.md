# GenORM

SQL Builder to prevent SQL mistakes using the Golang generics

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
