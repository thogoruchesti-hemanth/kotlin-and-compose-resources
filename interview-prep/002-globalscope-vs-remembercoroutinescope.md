# 🎙️ Interview Question: GlobalScope vs rememberCoroutineScope in Jetpack Compose

> **Track:** Android Engineering Interview Prep  
> **Topic:** Coroutine Scopes, Lifecycle-Aware Cancellation, Compose Memory Leaks  
> **Author:** [Thogaruchesti Hemanth](https://www.linkedin.com/in/thogoruchesti-hemanth/)

## ❓ The Interview Question

> *"If you launch a coroutine inside a composable using `GlobalScope` instead of `rememberCoroutineScope`, what's the actual risk?"*

## ❌ The Common Answer (and why it falls short)

**What candidates often say:**  
> *"It should still work fine — the coroutine runs in the background and does its job either way."*

### Why this answer is incomplete:
It mistakes **execution** for **correctness**. The primary risk isn't whether the coroutine will execute; it's **when and how it stops**. 

`GlobalScope` is an application-wide scope that lives for the entire process lifetime. It has no tie to the Compose runtime or any UI lifecycle. If a screen composable leaves composition (e.g., the user presses back or navigates away), any work launched in `GlobalScope` continues executing indefinitely in the background.

## 💡 The Strong Answer (Senior / Staff Level)

**What interviewers want to hear:**  
> *"While `GlobalScope` will technically execute the coroutine, it introduces significant lifecycle and memory risks. `GlobalScope` is decoupled from the Compose composition lifecycle.*
> *If the user navigates away before the task finishes, the coroutine continues to run. If that coroutine captures references to view models, callbacks, or state held by the composable, it leaks memory. Furthermore, if it tries to update UI state after the composable has left composition, it can lead to silent errors, wasted work, or crashes.*
> *`rememberCoroutineScope` avoids this completely because the returned `CoroutineScope` is bound to the call site's presence in the composition tree — it automatically cancels all child jobs the moment the composable leaves the tree."*

## 💻 Code Comparison

### ❌ The Risky Approach (`GlobalScope`)
```kotlin
@Composable
fun ProfileScreen(userId: String) {
    // ⚠️ ANTI-PATTERN: GlobalScope has no UI lifecycle awareness
    Button(onClick = {
        GlobalScope.launch {
            val details = fetchUserDetails(userId)
            // If the user navigated away during fetchUserDetails(),
            // this coroutine continues running, consuming network & CPU,
            // and holds onto references tied to this destroyed screen.
        }
    }) {
        Text("Fetch Details")
    }
}

```

### ✅ The Safe Approach (`rememberCoroutineScope`)

```kotlin
@Composable
fun ProfileScreen(userId: String) {
    // Bound to this composable's presence in the composition
    val scope = rememberCoroutineScope()

    Button(onClick = {
        scope.launch {
            val details = fetchUserDetails(userId)
            // If the user navigates back while this is in flight,
            // this job is automatically canceled cleanly.
        }
    }) {
        Text("Fetch Details")
    }
}

```

## 📊 Feature Comparison

| Dimension | `GlobalScope` | `rememberCoroutineScope()` |
| --- | --- | --- |
| **Lifecycle Bound** | Process / Application lifetime | Point of call in Composition |
| **Automatic Cancellation** | ❌ Never (manual cancellation required) | ✅ Automatic when composable leaves tree |
| **Memory Leak Risk** | High (captures closures/references) | Low / Cleaned up on leave |
| **Intended Use Case** | True top-level, app-wide operations | User interactions / event handlers in UI |

## 🧠 Core Interview Takeaway

> *"In Jetpack Compose UI, all asynchronous work triggered by user interaction must be tied to a bounded lifecycle. Use `rememberCoroutineScope` for event callbacks (like `onClick`) and `LaunchedEffect` for state-driven side effects."*

### Connect

* **LinkedIn:** [in/thogoruchesti-hemanth](https://www.google.com/url?sa=E&source=gmail&q=https://www.linkedin.com/in/thogoruchesti-hemanth/)
* **Repository:** ⭐ Star this repo if you find these Compose interview deep-dives helpful!
