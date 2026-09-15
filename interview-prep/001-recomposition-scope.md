# 🎙️ Interview Question: What Actually Recomposes in Jetpack Compose?

> **Topic:** Jetpack Compose Runtime, Recomposition Scope, and Stability  
> **Author:** [Thogaruchesti Hemanth](https://www.linkedin.com/in/thogoruchesti-hemanth/)

## ❓ The Interview Question

> *"When a piece of state changes in Compose, what actually gets recomposed?"*

## ❌ The Common Answer (and why it falls short)

**What candidates typically say:**
> *"The screen recomposes because state changed, redrawing the UI with the new value."*

### Why this answer falls short:
This answer describes the **trigger**, not the **scope**. Compose does not recompose "the whole screen." Assuming the entire screen recomposes ignores how the Compose runtime tracks reads and schedules targeted updates.


## 💡 The Stronger Answer (Senior / Staff Level)

**What interviewers look for:**
> *"Compose tracks which composable functions read a given piece of state during composition. When that state changes, only those specific composables are scheduled for recomposition — not their parents or unaffected siblings — provided their other inputs are stable. Everything else in the tree is skipped entirely."*

## 🔍 Deep Dive: Scope vs Trigger

Compose divides your UI tree into discrete **recomposition scopes**.

1. **State Observation Tracking:** When a composable reads a `State<T>` object (such as via `mutableStateOf`), Compose records that read inside the slot table for that specific scope.
2. **Granular Scheduling:** When a write happens to `.value`, only scopes that previously read that value are marked invalid.
3. **Smart Skipping:** If a composable’s parameters are stable and haven't changed, Compose skips executing its body completely, reusing the previously emitted UI node.

## 💻 Code Demonstration

```kotlin
@Composable
fun CounterScreen() {
    var count by remember { mutableStateOf(0) }

    Column(modifier = Modifier.padding(16.dp)) {
        // ❌ Does NOT recompose when 'count' changes
        HeaderTitle(title = "Counter Screen")

        // ✅ RECOMPOSES: Directly reads 'count'
        CounterDisplay(count = count)

        // ❌ Does NOT recompose: Button lambdas and static text are stable
        Button(onClick = { count++ }) {
            Text("Increment")
        }
    }
}

@Composable
fun HeaderTitle(title: String) {
    // Skipped during increment
    Text(text = title, style = MaterialTheme.typography.titleLarge)
}

@Composable
fun CounterDisplay(count: Int) {
    // Recomposes because its parameter changes
    Text(text = "Count: $count")
}

```

## ⚠️ The Stability Gotcha

Recomposition scoping only works cleanly if parameters are **stable**:

* **Stable parameters:** Primitives, `String`, function types, and data classes whose properties are all `val` and stable.
* **Unstable parameters:** Standard mutable collections (like `List<T>` without `@Immutable` or immutable collections) can cause Compose to assume inputs changed, widening recomposition beyond what was necessary.

## 📊 Comparison Summary

| Aspect | The Common Answer | The Stronger Answer |
| --- | --- | --- |
| **Focus** | Framework trigger ("state changed") | Execution scope ("which functions read it") |
| **Granularity** | Screen-level / holistic view | Nearest recomposition scope |
| **Optimization** | Assumes full re-render | Leverages skipping with stable inputs |


## 🧠 Core Interview Takeaway

> *"State changes don't invalidate entire screens — they invalidate only the nearest recomposition scopes that read the changed state, skipping everything with stable, unchanged inputs."*
