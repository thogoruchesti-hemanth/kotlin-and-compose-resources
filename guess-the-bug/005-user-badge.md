# Guess the Bug 5

This code looks fine. It isn't.

```kotlin
data class User(
    var name: String,
    val id: String
)

@Composable
fun UserBadge(user: User) {
    Text(text = user.name)
}

```

## What breaks, and When?

# Here's what's happening:

Two critical issues break Compose's core reactivity model here:

1. **Mutations don't notify Compose**: `User.name` is a mutable property (`var`). If `user.name = "New Name"` is modified outside, Compose has no mechanism to observe this mutation because it isn't backed by Compose `State`. The UI will silently fail to update.
2. **Class instability prevents skipping**: Because `User` contains a mutable property (`var name`), the Compose compiler flags the entire `User` class as **unstable**. Even when `user` hasn't changed at all, if the parent composable recomposes, `UserBadge` will be forced to recompose every single time instead of skipping.

Fix: Make `User` immutable by using `val` for all properties. When updates are required, emit a new instance via `.copy(name = ...)` or use Compose `State`. This guarantees stability, enables composable skipping, and triggers updates reliably.

## Before Code:

```kotlin
data class User(
    var name: String,
    val id: String
)

@Composable
fun UserBadge(user: User) {
    Text(text = user.name)
}

```

## After Code:

```kotlin
data class User(
    val name: String,
    val id: String
)

@Composable
fun UserBadge(user: User) {
    Text(text = user.name)
}

```

*(When updating the user state in your ViewModel or caller, trigger state updates immutably using `.copy()`: `user = user.copy(name = "Updated Name")`)*
