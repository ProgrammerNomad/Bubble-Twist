# Account Functionality: Technical Overview

This document outlines how account features are implemented in Bubble-Twist.

## Data Storage
Bubble-Twist uses the `shared_preferences` package to store user data on the device. Data such as username, user ID, password, and activity timestamps are stored as key-value pairs. This ensures persistence even after closing the app.

### Keys Used
- `USERNAME`: Stores the player's username
- `ID`: Stores the player's unique user ID
- `PASS`: Stores the user's password
- `lastClearedDaily`, `lastClearedWeekly`, `lastClearedMonthly`: Timestamps for the last cleared daily/weekly/monthly activity
- `leaderboardConcent`: Stores user's consent status for leaderboard participation

## Example Flow
1. **On User Registration or Login:**
   - Username, ID, and password are saved using shared preferences.
2. **Activity Completion:**
   - When a user completes a daily, weekly, or monthly task, the corresponding timestamp is updated.
3. **Leaderboard Consent:**
   - The app checks or updates the user's consent status for leaderboard participation as needed.

## Example Code Snippet
```dart
// Save username
await SharedPreferencesHelper.setUsername('player1');
// Retrieve username
String username = await SharedPreferencesHelper.getUsername();
```

All accesses are asynchronous to ensure smooth app performance. For more usage details, see [usage.md](usage.md).