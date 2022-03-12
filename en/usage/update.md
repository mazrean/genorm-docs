# Update

SQL UPDATE statements can be executed using the `genorm.Update` function.

It must start with a `genorm.Update` function and end with either `Do` or `DoCtx`, but the rest of the order can be swapped. However, if the same method other than `OrderBy` is executed twice in the same method chain, `Do` and `DoCtx` will return an error without querying at runtime.

#### Example

```go
// UPDATE `users` SET `name`="name"
affectedRows, err = genorm.
    Update(orm.User()).
    Set(
        genorm.AssignLit(user.Name, genorm.Wrap("name")),
    ).
    Do(db)

// UPDATE `users` SET `name`="name"
affectedRows, err = genorm.
    Update(orm.User()).
    Set(
        genorm.AssignLit(user.Name, genorm.Wrap("name")),
    ).
    DoCtx(context.Background(), db)

// UPDATE `users` SET `name`="name" WHERE `id`={{uuid.New()}} 
affectedRows, err = genorm.
    Update(orm.User()).
    Set(
        genorm.AssignLit(user.Name, genorm.Wrap("name")),
    ).
    Where(genorm.EqLit(user.IDExpr, uuid.New())).
    Do(db)

// UPDATE `users` SET `name`="name" ORDER BY `created_at` LIMIT 1
affectedRows, err = genorm.
    Update(orm.User()).
    Set(
        genorm.AssignLit(user.Name, genorm.Wrap("name")),
    ).
    OrderBy(genorm.Desc, user.CreatedAt).
    Limit(1).
    Do(db)

// UPDATE `users` INNER JOIN `messages` ON `users.id` = `messages`.`id` SET `content`="hello world"
userIDColumn := orm.MessageUserParseExpr(user.ID)
messageUserIDColumn := orm.MessageUserParseExpr(message.UserID)
messageContent := orm.MessageUserParse(message.Content)
affectedRows, err := genorm.
  Update(orm.User().
		Message().Join(genorm.Eq(userID, messageUserID))).
  Set(genorm.AssignLit(messageContent, genorm.Wrap("hello world"))).
  Do(db)
```

### Update

Function to start a chain of methods to issue `UPDATE` statements. It takes a table generated by the CLI as its first argument.

### Set(required)

Sets a value set of columns whose values are to be updated with `UPDATE`. It takes a variable-length argument of type `*TableAssignExpr[T]`. Values of type `*TableAssignExpr[T]` are obtained as the return value of the `genorm.Assign` or `genorm.AssignLit` function.

### Where(optional)

Set the `WHERE` clause in the `UPDATE` statement. The first argument takes an expression corresponding to `bool` in the Golang.

### Do

Executes the query and exits the method chain. The argument is a value satisfying the `genorm.DB`interface including [`*sql.DB`](https://pkg.go.dev/database/sql#DB)/[`*sql.Tx`](https://pkg.go.dev/database/sql#Tx) It takes this and uses it to execute the query.

### DoCtx

Executes the query and exits the method chain. The second argument is a value satisfying the `genorm.DB`interface including [`*sql.DB`](https://pkg.go.dev/database/sql#DB)/[`*sql.Tx`](https://pkg.go.dev/database/sql#Tx) It takes this and uses it to execute the query. It also receives `context.Context` as its first argument. When context is canceled, the connection to the database is released, preventing unnecessary use of connections.