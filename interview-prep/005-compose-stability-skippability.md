# 🎙️ Interview Question: Unchanged Data, Constant Recompositions

> **Track:** Android Engineering Interview Prep
> 
> 
> **Topic:** Jetpack Compose Stability, Skippability, Compose Compiler Metrics
> 
> 
> **Author:** [Thogaruchesti Hemanth](https://www.linkedin.com/in/thogoruchesti-hemanth/?utm_source=gemini)
> 

## ❓ The Interview Question

> *"A composable recomposes every time its parent does, even though the object it receives hasn't changed. What could cause that?"*
> 

## ❌ The Common Answer (and why it fails)

**What candidates often say:**

> *"Maybe the data class doesn't override equals() correctly, so Compose thinks values changed."*
> 

### Why this answer falls short:

If the parameter type is unstable, Compose never even reaches an equality check. Skippability is lost at compile time regardless of `equals()`. Compose only compares values using `equals()` at runtime if the parameter type is marked as **stable**.

## 💡 The Strong Answer (Staff / Senior Level)

**What interviewers want to hear:**

> *"This is caused by parameter instability making the composable non-skippable. At compile time, the Compose compiler infers whether types are stable or unstable. If even one parameter is inferred as unstable, the compiler treats the composable as restartable but NOT skippable.*
> *Hidden culprits that cause instability include:*
> 1. *Using standard collection interfaces like `List<T>`, `Set<T>`, or `Map<T>` (interfaces cannot guarantee immutability).*
> 2. *Classes with mutable properties (`var`).*
> 3. *Classes defined in non-Compose external Gradle modules or third-party libraries.*
> 
> 
> *To fix this, we can use Kotlinx `ImmutableList`, annotate external/unstable classes with `@Immutable` or `@Stable`, wrap collections in a data class with `@Immutable`, or enable Strong Skipping Mode."*


## 💻 Code Breakdown

### ❌ Problematic Code (Unstable Parameter Type)

```kotlin
// Data class containing a standard List (interface inferred as unstable)
data class UserProfile(
    val id: String,
    val name: String,
    val tags: List<String> // ⚠️ List interface is unstable in Compose
)

@Composable
fun UserCard(profile: UserProfile) {
    // Non-skippable: Recomposes on every parent recomposition,
    // even when 'profile' has the exact same content.
    Text(text = profile.name)
}

```

### ✅ Optimized Code (Guaranteed Stability & Skippability)

#### Approach 1: Using Kotlinx Immutable Collections

```kotlin
import kotlinx.collections.immutable.ImmutableList

@Immutable
data class UserProfile(
    val id: String,
    val name: String,
    val tags: ImmutableList<String>
)

@Composable
fun UserCard(profile: UserProfile) {
    // Skippable: Compose skips this composable if 'profile' equals the previous instance
    Text(text = profile.name)
}

```

#### Approach 2: Enabling Strong Skipping Mode (Compose Compiler 1.5.4+ / Kotlin 2.0+)

With Strong Skipping enabled in your Gradle configuration, Compose compares unstable parameters by instance equality (`===`), making composables skippable by default without manual `@Immutable` wrappers.

## 📊 Stability & Skippability Matrix

| Class Type | Compiler Tag | Skippable when unchanged? | `equals()` Checked? |
| --- | --- | --- | --- |
| **Primitives (`Int`, `String`)** | `@Stable` | **Yes** | Yes (equality check) |
| **`data class` with all `val` & stable types** | Stable | **Yes** | Yes (structural equality) |
| **`data class` with `var**` | Unstable | **No** | Never reached

 |
| **Standard `List<T>` / `Map<K, V>**` | Unstable | **No** | Never reached

 |
| **`ImmutableList<T>`** | Stable | **Yes** | Yes (structural equality) |


## 🛠️ Verification via Compose Compiler Metrics

To inspect why composables are failing to skip:

1. Enable compiler metrics in your `build.gradle.kts`:
```kotlin
composeCompiler {
    reportsDestination = layout.buildDirectory.dir("compose_metrics")
    metricsDestination = layout.buildDirectory.dir("compose_metrics")
}

```


2. Build the project and inspect `<module>-composables.txt`:
* Look for `restartable skippable fun UserCard(...)` vs `restartable fun UserCard(...)`.
* Look for `unstable profile: UserProfile` in the parameters list.



## 🎯 The Rule of Thumb for Compose Interviews

> **`equals()` only prevents recomposition if the compiler has already decided the parameter type is stable. If a type is unstable, Compose ignores `equals()` completely and always recomposes.**


### Connect

* **LinkedIn:** [in/thogoruchesti-hemanth](https://www.linkedin.com/in/thogoruchesti-hemanth/?utm_source=gemini)

* **Repository:** Star this repo for more Jetpack Compose interview challenges and deep-dives! ⭐
