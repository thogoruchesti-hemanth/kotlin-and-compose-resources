# Guess the Bug 3

This code looks fine. It isn't.

```kotlin
@Composable
fun ShoppingList() {
    val items = remember { mutableListOf<String>() }

    Column {
        Button(onClick = {
            items.add("Item ${items.size + 1}")
        }) {
            Text("Add item")
        }
        items.forEach { Text(it) }
    }
}

```

## What breaks, and When?

# Here's what's happening:

Tapping the button does add a new string to the list — but nothing on screen updates. Compose only tracks changes to `State<T>` objects it's reading; a plain `MutableList` is just a regular Kotlin object. Mutating its contents doesn't notify Compose of anything, because the reference held by `remember` never actually changes. The list grows internally, but the UI has no idea.

Fix: use `remember { mutableStateListOf<String>() }` instead of a plain `mutableListOf`. It's a snapshot-backed list that Compose observes directly, so additions and removals trigger recomposition on their own.

## Before Code:

```kotlin
@Composable
fun ShoppingList() {
    val items = remember { mutableListOf<String>() }

    Column {
        Button(onClick = {
            items.add("Item ${items.size + 1}")
        }) {
            Text("Add item")
        }
        items.forEach { Text(it) }
    }
}

```

## After Code:

```kotlin
@Composable
fun ShoppingList() {
    val items = remember { mutableStateListOf<String>() }

    Column {
        Button(onClick = {
            items.add("Item ${items.size + 1}")
        }) {
            Text("Add item")
        }
        items.forEach { Text(it) }
    }
}

```
