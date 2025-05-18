# 12-Week Year Progress Dataview for Obsidian

A DataviewJS script for Obsidian that helps users track progress on multiple projects and habits over a specified time period, inspired by the 12-Week Year methodology.

## Features

- **Project Progress Tracking**: Track completion of multiple projects with weighted tasks
- **Weekly Progress Visualization**: See your progress week by week with visual indicators
- **Habit Consistency Tracking**: Monitor your habits with weekly consistency metrics
- **Overall Habit Progress**: View average consistency across all tracked weeks
- **Visual Progress Bars**: Native Obsidian/Dataview progress bars for all metrics
- **Customizable Weights**: Prioritize certain projects with custom weighting
- **Date Range Support**: Specify custom start and end dates for your 12-week period
- **Note Link Support**: Use Obsidian note links for dates and project/habit names (e.g., `[[2025-05-12]]`, `[[Project Name]]`)
- **Debug Mode**: Optional debug information for troubleshooting

## Setup

1. Install the [Dataview plugin](https://github.com/blacksmithgu/obsidian-dataview) for Obsidian
2. Create a `goals.md` file in your vault with your projects and habits (see format below)
3. Create a note where you want to display your progress
4. Add the following code block to that note:

````markdown
```dataviewjs
await dv.view("progress_dataview")
```
````

5. Place the `progress_dataview.js` file in your Obsidian vault's `.obsidian/views/` folder

## Data Structure

### Goals File Format (goals.md)

```markdown
---
aliases: [Goals]
---

- Start:: [[2025-05-12]]
- End:: [[2025-07-17]]
- Debug:: true

# My Projects

- project:: laser appointment
- weight:: 2
- tasks:: 4
- project:: another_project_name
- tasks:: 7

# My Habits

- habit:: meditate
- weekly:: 4
- habit:: gym
- weekly:: 3
```

### Daily Notes Format

Tasks in daily notes should exactly match the project or habit names:

```markdown
- [x] laser appointment
- [x] another_project_name
- [x] meditate
- [x] gym
```

## Output

The script generates several tables:

1. **Project Progress**: Shows overall progress for each project
   - Project name
   - Total tasks (with weight)
   - Progress percentage
   - Progress bar

2. **Weekly Progress**: Shows progress for each week
   - Week number
   - Date range
   - Progress percentage
   - Progress bar

3. **Habit Scorecard**: Shows habit consistency for each week
   - Week number
   - Date range
   - Habit name
   - Consistency percentage
   - Progress bar

4. **Overall Habit Progress**: Shows average consistency across all weeks
   - Habit name
   - Consistency average (average weekly count / weekly target with percentage)
   - Progress bar

## Customization

- **Debug Mode**: Set `Debug:: true` in your goals.md file to see detailed information
- **Date Range**: Customize the start and end dates in your goals.md file
- **Project Weights**: Assign different weights to prioritize certain projects
- **Weekly Habit Targets**: Set custom weekly targets for each habit

## Future Improvements

- Support for note links with aliases (e.g., `[[Project Name|Alias]]`)
- Historical weekly progress tracking
- Burndown chart visualization
- Project categories or tags
- Project deadlines and priority levels
- More flexible task matching system
- Dashboard view with multiple visualizations
- Support for recurring projects
- Notifications for projects that are behind schedule

## License

This project is open source and available under the MIT License.
