# Automatic Past Session Skipping

This document explains the newly implemented automatic past session skipping feature in TimePilot.

## Overview

Sessions that are not yet completed but the day has passed are now automatically marked as skipped. These sessions will not be redistributed to future dates, maintaining the app's "forward focus" approach.

## How It Works

### 1. Automatic Detection
- When the app loads, it checks all study plans for sessions scheduled on dates before today
- Sessions that are still marked as 'scheduled' (not completed, skipped, or done) are automatically marked as 'skipped'
- This happens both on initial app load and daily when the date changes

### 2. Skipping Behavior
- **Status Change**: Past incomplete sessions get `status: 'skipped'`
- **Metadata**: Each automatically skipped session gets skip metadata:
  ```typescript
  skipMetadata: {
    skippedAt: new Date().toISOString(),
    reason: 'overload' // Indicates system auto-skip due to date passing
  }
  ```

### 3. No Redistribution
- Automatically skipped sessions are treated the same as manually skipped sessions
- They are excluded from:
  - Study plan regeneration
  - Session redistribution
  - Workload calculations
  - Calendar display (filtered out)
  - Task completion calculations

### 4. User Notification
- When sessions are automatically skipped, users see a notification:
  ```
  📅 X past incomplete session(s) were automatically marked as skipped. They will not be rescheduled.
  ```

## Implementation Details

### Core Function
```typescript
// Located in src/utils/scheduling.ts
export const markPastSessionsAsSkipped = (studyPlans: StudyPlan[]): StudyPlan[]
```

This function:
- Iterates through all study plans
- Checks if plan date is before today
- Marks incomplete sessions as skipped with appropriate metadata
- Returns updated study plans

### Integration Points

1. **App Load Check** (`src/App.tsx`)
   - Runs once after initial data load
   - Uses ref to prevent multiple executions
   - Shows notification for newly skipped sessions

2. **Daily Check** (`src/App.tsx`)
   - Uses localStorage to track last check date
   - Runs when app loads on a new day
   - Silently processes past sessions without notification

### Existing Compatibility

The feature integrates seamlessly with existing functionality:

- **Session Status**: Builds on existing status system ('scheduled', 'completed', 'skipped')
- **Skip Metadata**: Uses existing SkipMetadata interface
- **Filtering**: Leverages existing `filterSkippedSessions()` utility
- **Forward Focus**: Maintains the app's forward-looking philosophy

## User Experience

### Before This Feature
- Past incomplete sessions remained as 'scheduled' indefinitely
- Users might see outdated sessions from previous days
- Manual intervention required to handle missed sessions

### After This Feature
- Past sessions are automatically cleaned up
- Users only see relevant current and future sessions
- Clear indication when sessions are auto-skipped
- Maintains schedule hygiene automatically

## Benefits

1. **Automatic Cleanup**: No manual intervention needed for past sessions
2. **Clear Schedule**: Users see only relevant sessions
3. **Forward Focus**: Encourages looking ahead rather than dwelling on missed sessions
4. **Consistent State**: Prevents accumulation of outdated 'scheduled' sessions
5. **No Redistribution**: Prevents overwhelming future schedules with old sessions

## Technical Notes

- The function is efficient - only processes plans with dates before today
- Uses JSON comparison to detect actual changes before updating state
- Notification counting uses timestamp checking to avoid showing notifications for old auto-skipped sessions
- Daily check uses localStorage for persistence across app sessions
- Compatible with all existing session management features

## Future Enhancements

Potential improvements could include:
- User preference to disable auto-skipping
- Different skip reasons (e.g., 'auto_missed', 'day_passed')
- Analytics on auto-skipped sessions
- Optional redistribution with user confirmation
