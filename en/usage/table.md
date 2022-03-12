# Table

GenORM passes the table as a value at the beginning of the method chain that executes the SQL statement.

## Normal Table
Values representing normal tables that do not contain joins can be obtained by calling the function of the same name as the structure name of the configuration file in the generated code.
For example, the `users` table used in the code generation example has a configuration file struct name of `User`, so the value corresponding to the table can be obtained by calling the `User` function in the generated code. In other words, if the value of the `package` flag specified during code generation is `orm`, then `orm.User()` is the value corresponding to the `users` table.

### Example
```go
// `users` table
orm.User()

// `messages` table
orm.Message()
```

## Joined Table
A table created by performing a Join, such as `users INNER JOIN messages ON users.id = messages.user_id`, is a table that is created by joining a value (`orm.User()`) representing the source table (`users`) with the value of `orm.User()`. It can be created by calling the method corresponding to the table and then the method (`Join`) corresponding to the type of Join.
The method corresponding to the Join type also takes an ON clause condition as an argument.

### Example
```go
// Column type conversion is necessary because the post-join table and the pre-join table are different.
userID := orm.MessageUserParseExpr(user.ID)
messageUserID := orm.MessageUserParseExpr(message.UserID)

// `users INNER JOIN messages ON users.id = messages.user_id`
innerJoinedTable := orm.User().
	Message().Join(genorm.Eq(userID, messageUserID))

// `users CROSS JOIN messages
crossJoinedTable := orm.User().
	Message().Join(nil)

// `users LEFT JOIN messages ON users.id = messages.user_id`
leftJoinedTable := orm.User().
	Message().LeftJoin(genorm.Eq(userID, messageUserID))

// `users RIGHT JOIN messages ON users.id = messages.user_id`
rightJoinedTable := orm.User().
	Message().RightJoin(genorm.Eq(userID, messageUserID))
```
