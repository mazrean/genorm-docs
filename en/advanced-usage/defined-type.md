# Defined Type

Using a Defined Type that satisfies the `genorm.ExprType`interface is an even stronger way to prevent SQL mistakes.

In the Configuration file used as an example of usage, both the `id` and `user_id` columns of the `messages` table are of type `uuid.UUID`. Therefore, the

```go
messageValues, err := genorm.
	Select(orm.Message()).
	Where(genorm.Eq(message.IDExpr, message.UserIDExpr)).
	GetAll(db)
```

will not result in a compile error.

However, there should be few cases where you want to compare the id of a message with the id of a user.

Define the `MessageID` and `UserID` types as follows.

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

Then, set the `id` and `user_id` columns in the `messages` table in the Configuration file to the `MessageID` and `UserID` types.

By doing so, message's id and user's id can be treated as different types, and the following code will result in a compile error.

```go
messageValues, err := genorm.
	Select(orm.Message()).
	Where(genorm.Eq(message.IDExpr, message.UserIDExpr)).
	GetAll(db)
```
