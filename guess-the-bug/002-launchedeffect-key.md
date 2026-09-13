# Guess the Bug 2

This code looks fine. It isn't.

```kotlin
@Composable
fun UserProfileScreen(
    userId: String,
    viewModel: ProfileViewModel = viewModel()
) {
    val profile by viewModel.profile.collectAsState()

    LaunchedEffect(Unit) {
        viewModel.loadProfile(userId)
    }

    ProfileContent(profile)
}

```

## What breaks, and When?

# Here's what's happening:

`LaunchedEffect(Unit)` only runs once for as long as this composable stays in the composition. If the screen navigates from one user's profile to another without leaving composition — a common pattern in master-detail or tabbed layouts — `userId` changes, but the effect never restarts. `loadProfile` only ever runs for the first user ID it saw, and the screen keeps showing stale data for every profile after that.

Fix: key the effect on the value it depends on — `LaunchedEffect(userId)` — so it restarts whenever `userId` actually changes.

## Before Code:

```kotlin
@Composable
fun UserProfileScreen(
    userId: String,
    viewModel: ProfileViewModel = viewModel()
) {
    val profile by viewModel.profile.collectAsState()

    LaunchedEffect(Unit) {
        viewModel.loadProfile(userId)
    }

    ProfileContent(profile)
}

```

## After Code:

```kotlin
@Composable
fun UserProfileScreen(
    userId: String,
    viewModel: ProfileViewModel = viewModel()
) {
    val profile by viewModel.profile.collectAsState()

    LaunchedEffect(userId) {
        viewModel.loadProfile(userId)
    }

    ProfileContent(profile)
}

```
