# Guess the Bug 4

This code looks fine. It isn't.

```kotlin
@Composable
fun ScrollToTopButton(listState: LazyListState) {
    val showButton = listState.firstVisibleItemIndex > 0

    AnimatedVisibility(visible = showButton) {
        FloatingActionButton(onClick = { /* scroll to top */ }) {
            Icon(
                Icons.Default.ArrowUpward,
                contentDescription = null
            )
        }
    }
}

```

## What breaks, and When?

# Here's what's happening:

While scrolling through the list, the composable triggers frequent, unnecessary recompositions on every single item index change. Because `listState.firstVisibleItemIndex` is read directly inside the composable's body, any change to the scroll index forces the entire `ScrollToTopButton` composable to recompose—even though the boolean visibility state only actually flips when transitioning between index `0` and index `> 0`. During fast scrolling, this causes redundant work and UI jank.

Fix: wrap the calculation in `remember { derivedStateOf { ... } }`. `derivedStateOf` observes the rapidly changing `firstVisibleItemIndex`, but only triggers recomposition when the resulting boolean value (`showButton`) actually flips from `true` to `false` or vice versa.

## Before Code:

```kotlin
@Composable
fun ScrollToTopButton(listState: LazyListState) {
    val showButton = listState.firstVisibleItemIndex > 0

    AnimatedVisibility(visible = showButton) {
        FloatingActionButton(onClick = { /* scroll to top */ }) {
            Icon(
                Icons.Default.ArrowUpward,
                contentDescription = null
            )
        }
    }
}

```

## After Code:

```kotlin
@Composable
fun ScrollToTopButton(listState: LazyListState) {
    val showButton by remember {
        derivedStateOf { listState.firstVisibleItemIndex > 0 }
    }

    AnimatedVisibility(visible = showButton) {
        FloatingActionButton(onClick = { /* scroll to top */ }) {
            Icon(
                Icons.Default.ArrowUpward,
                contentDescription = null
            )
        }
    }
}

```
