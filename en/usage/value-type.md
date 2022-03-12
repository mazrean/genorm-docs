# Value Type

GenORM categorizes SQL expressions into the following three types, and prevents SQL errors by mapping each SQL expression to a Go type.

* Literal
* Expr
* Column

Also, "the type itself implements [sql.Scanner](https://pkg.go.dev/database/sql#Scanner) and the pointer implements [driver.Valuer](https://pkg.go.dev/database/sql/driver#Valuer)" such as `uuid.UUID` can be placed in the `genorm.ExprType`interface.

### Literal

This value is treated as a literal in SQL.

The value of the type that applies to the `genorm.ExprType`interface is literal.

Values of primitive types such as `bool`, `int`, and `float32` that cannot be placed in the `genorm.ExprType` interface can be converted to the `genorm.WrappedPrimitive[T]` type using the `genorm. It can be used as a literal.

### Expr

This is a value that is handled as an expression in SQL.

This value has as its type parameter the type of the table used in constructing the expr and the type that applies to the `genorm.ExprType`interface to which the expr corresponds.

For example, the return value of `genorm.EqLit(user.IDExpr, uuid.New())` is an Expr with `*orm.UserTable` and `bool` as type parameters because it uses the id column of the users table and is a boolean in SQL.

Normal table columns can be used as `Expr` by using a variable named `~Expr` in the package corresponding to the table, such as `users.IDExpr` for an `id` column in a `users` table.
Columns of a joined table can be converted to `Expr` of the table using the function `~ParseExpr` corresponding to the table.
For example, if `users INNER JOIN messages ON users.id = messages.user_id`, you can use the `id` column of the `users` table by using `orm.MessageUserParseExpr(user.ID)`.

### Column

This value is treated as a column name in SQL.

This value has as type parameters the type corresponding to the table containing the column and the type that applies to the `genorm.ExprType`interface to which Expr corresponds.

Normal table columns can be used as `columns` by using variables in the package corresponding to the table, such as `users.ID` for an `id` column in the `users` table.
Columns of the joined table can be converted to `columns` of the table using the function `~Parse` corresponding to the table.
For example, if `users INNER JOIN messages ON users.id = messages.user_id`, then `orm.MessageUserParse(user.ID)` can be used as `Column` for the `id` column of the `users` table.
