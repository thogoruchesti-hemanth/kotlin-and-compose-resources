# 🎙️ Interview Question: Diagnosing Scroll Jank & Recomposition Frequency

> **Track:** Android Engineering Interview Prep  
> **Topic:** Jetpack Compose Performance, `derivedStateOf` vs `remember`, State Observation Scope  
> **Author:** [Thogaruchesti Hemanth](https://www.linkedin.com/in/thogoruchesti-hemanth/)

## ❓ The Interview Question

> *"A list screen has a floating action button that shows or hides based on scroll position. Users on low-end devices report noticeable jank while scrolling. Where would you look first?"*


## ❌ The Common Answer (and why it fails)

**What candidates often say:**  
> *"I'd check whether the visibility calculation is wrapped in `remember` — forgetting `remember` usually causes unnecessary recomposition."*

### Why this answer falls short:
`remember` prevents an object or calculation from being re-instantiated across recomposition passes, but **it does not control how often recomposition is triggered in the first place**. 

```kotlin
// ❌ Still causes jank:
val showButton = remember(listState.firstVisibleItemIndex) {
    listState.firstVisibleItemIndex > 0
}

```

If the expression reads a high-frequency input like `firstVisibleItemIndex` directly, that input fires on nearly every scroll frame. Wrapping it in `remember` does not dampen recomposition frequency at all because the key invalidates continuously.


## 💡 The Strong Answer (Staff / Senior Level)

**What interviewers want to hear:**

> *"I'd inspect what the visibility calculation actually reads during composition. If it reads raw scroll state like `firstVisibleItemIndex` directly, the composable is scheduled for recomposition on every scroll frame rather than only when the resulting boolean changes.*
> *I would wrap the comparison in `derivedStateOf` to buffer high-frequency state reads into a low-frequency boolean emission, gating recomposition purely on state transitions. I'd then verify the improvement using Layout Inspector recomposition counters."*


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
| **Device Impact** | Dropped frames on low-end hardware | Stable 60/120 FPS |

## 🛠️ Verification via Android Studio Layout Inspector

1. Deploy the app in profileable or debug mode.
2. Open **Layout Inspector** (`View > Tool Windows > Layout Inspector`).
3. Turn on **Show Recomposition Counts**.
4. Scroll through the list:
* **Direct Read:** Count increments continuously per scroll gesture.
* **With `derivedStateOf`:** Count increments only twice (when passing item 0 in either direction).

## 🎯 The Rule of Thumb for Compose Interviews

> **Use `derivedStateOf` whenever a calculation reads state that changes significantly more frequently than the output value your UI actually needs to consume.**


### Connect

* **LinkedIn:** [in/thogoruchesti-hemanth](https://www.google.com/url?sa=E&source=gmail&q=https://www.linkedin.com/in/thogoruchesti-hemanth/)
* **Repository:** Star this repo for more Jetpack Compose interview questions and system deep-dives! ⭐
