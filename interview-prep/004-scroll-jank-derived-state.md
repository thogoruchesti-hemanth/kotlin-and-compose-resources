# 🎙️ Interview Question: Diagnosing Scroll Jank on Low-End Devices

> **Track:** Android Engineering Interview Prep
> 
> 
> **Topic:** Jetpack Compose Performance, `derivedStateOf` vs `remember`, State Observation Scope
> 
> 
> **Author:** [Thogaruchesti Hemanth](https://www.linkedin.com/in/thogoruchesti-hemanth/?utm_source=gemini)
> 

## ❓ The Interview Question

> *"A FAB shows or hides on scroll position. Low-end devices report jank while scrolling. Where would you look first?"*

## ❌ The Common Answer (and why it fails)

**What candidates often say:**

> *"I'd check whether the visibility value is wrapped in `remember` — forgetting `remember` usually causes extra recomposition."*

### Why this answer falls short:

`remember` prevents recreation across recomposition, not frequency. If the calculation reads raw scroll position directly, the input still changes every frame.

```kotlin
// ❌ Still triggers recomposition continuously:
val showButton = remember(listState.firstVisibleItemIndex) {
    listState.firstVisibleItemIndex > 0
}

```

Passing high-frequency scroll state as a key to `remember` simply invalidates the block on practically every frame during a scroll gesture, keeping the composable scheduled for recomposition non-stop.

## 💡 The Strong Answer (Staff / Senior Level)

**What interviewers want to hear:**

> *"The exact mechanism causing this jank is reading high-frequency snapshot state directly inside the composition phase without buffering it. `firstVisibleItemIndex` changes on almost every frame during scrolling, forcing the calling composable to recompose continuously even though the target boolean output remains the same.*
> *To fix this, I wrap the condition in `derivedStateOf`. This buffers the high-frequency state changes and only invalidates the composition scope when the resulting boolean value actually flips from `false` to `true` or vice versa. I would then verify the fix by checking recomposition counts in the Android Studio Layout Inspector."*


## 💻 Code Breakdown

### ❌ Problematic Code (Unbuffered State Read)

```kotlin
@Composable
fun ScrollToTopButton(listState: LazyListState) {
    // Recomposes on every single index update during scrolling
    val showButton = listState.firstVisibleItemIndex > 0

    AnimatedVisibility(visible = showButton) {
        FloatingActionButton(onClick = { /* scroll to top */ }) {
            Icon(Icons.Default.ArrowUpward, contentDescription = "Scroll to top")
        }
    }
}

```

### ✅ Optimized Code (Buffered via `derivedStateOf`)

```kotlin
@Composable
fun ScrollToTopButton(listState: LazyListState) {
    // Recomposes ONLY when the result flips (true -> false or false -> true)
    val showButton by remember {
        derivedStateOf { listState.firstVisibleItemIndex > 0 }
    }

    AnimatedVisibility(visible = showButton) {
        FloatingActionButton(onClick = { /* scroll to top */ }) {
            Icon(Icons.Default.ArrowUpward, contentDescription = "Scroll to top")
        }
    }
}

```

## 📊 Performance Comparison

| Dimension | Direct Read | With `derivedStateOf` |
| --- | --- | --- |
| **Observation Type** | High-frequency raw index state | Low-frequency derived boolean |
| **Recompositions per 100-item scroll** | ~50–150+ recompositions | **2 recompositions total** |
| **Main Thread Budget** | Heavy layout & draw invalidation | Lightweight |
| **Device Impact** | Dropped frames & jank on low-end hardware

 | Smooth 60/120 FPS |


## 🛠️ Verification via Android Studio Layout Inspector

1. Run the app in debug or profileable mode on a device or emulator.
2. Open **Layout Inspector** (`View > Tool Windows > Layout Inspector`).
3. Turn on **Show Recomposition Counts**.
4. Scroll through the list:
* **Direct Read:** Count continuously ticks upward on every scroll increment.
* **With `derivedStateOf`:** Count only increments when crossing the threshold (item `0` to `> 0` and back).


## 🎯 The Rule of Thumb for Compose Interviews

> **Use `derivedStateOf` whenever a calculation reads state that changes significantly more frequently than the output value your UI actually needs to consume.**

### Connect

* **LinkedIn:** [in/thogoruchesti-hemanth](https://www.linkedin.com/in/thogoruchesti-hemanth/?utm_source=gemini)

* **Repository:** Star this repo for more Jetpack Compose interview challenges and performance breakdowns! ⭐
