# 12-week-year progress dashboard for Obsidian dataview

- [12-week-year progress dashboard for Obsidian dataview](#12-week-year-progress-dashboard-for-obsidian-dataview)
  - [Features](#features)
  - [Setup](#setup)
  - [Notes Structure](#notes-structure)
    - [Goals File Format (goals.md)](#goals-file-format-goalsmd)
    - [Daily Notes Format](#daily-notes-format)
  - [Output](#output)
  - [Parameters](#parameters)
  - [License](#license)

A DataviewJS script for Obsidian that helps users track progress on multiple projects and habits over a specified time period, inspired by the 12-Week Year book.

## Features

- **Project Progress Tracking**: Track completion of multiple projects with weighted tasks
- **Weekly Progress Visualization**: View weekly progress per project and overall
- **Habit Consistency Tracking**: Monitor habits with weekly consistency metrics
- **Overall Habit Progress**: View average consistency across all tracked weeks
- **Customizable Weights**: Prioritize certain projects with custom weighting
- **Date Range Support**: Specify custom start and end dates for your 12-week period
- **Note Link Support**: Use Obsidian note links for dates and project/habit names (e.g., `[[2025-05-12]]`, `[[Project Name]]`)

## Setup

1. Install the [Dataview plugin](https://github.com/blacksmithgu/obsidian-dataview) for Obsidian
2. Create a `goals.md` file in your vault with your projects and habits (see format below)
   - You can customixe that name, it is set in the beginning of the code block
3. Copy dashboard.md into your vault, or copy its contents into any existing note

## Notes Structure

### Goals File Format (goals.md)

```markdown

- Start:: [[2025-05-12]]
- End:: [[2025-07-17]]
- Debug:: true

# Projects and habits

- project:: [[foo]]
- weight:: 2
- tasks:: 4
- project:: bar
- tasks:: 7


- habit:: meditate
- weekly:: 4
- habit:: gym
- weekly:: 3
```

### Daily Notes Format

Completed tasks in daily notes matching the project names are counted:

```markdown
- [x] [[foo]]
- [x] bar
- [x] meditate
- [x] gym
```

## Output

![dashboard](/doc/dashboard_example.png)

## Parameters

- **Debug Mode**: Set `Debug:: true` in your goals.md file to see detailed information
- **Date Range**: Epoch (Quarter) Start and End dates.
- **Project Weights**: Assign different weights to prioritize certain projects in progress calculations
- **Weekly Habit Targets**: Set custom weekly targets for each habit

## License

MIT License.
