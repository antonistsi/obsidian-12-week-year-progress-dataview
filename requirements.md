# 12-Week Year Progress Dataview for Obsidian

## Overview
This project provides a DataviewJS script for Obsidian that helps users track progress on multiple projects over a specified time period. The script reads project definitions from a goals.md file, counts completed tasks in daily notes that match project names, and visualizes progress both overall and on a weekly basis.

## Requirements

### Core Functionality
- Track progress on multiple projects defined in a goals.md file
- Count completed tasks in daily notes that match project names
- Show overall progress for each project with visualization
- Provide weekly progress tracking against a calculated target
- Use date ranges specified in the goals.md file
- Support both direct dates and note links (e.g., [[2025-05-12]])
- Use native Obsidian/Dataview progress bars for visualization
- Support project weighting to prioritize certain projects
- List all weeks in the date range with strikethrough for past weeks
- Include optional debug mode

### Data Structure

#### Goals File Format (goals.md)
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
```



#### Daily Notes Format
Tasks in daily notes should exactly match the project names:
```markdown
- [x] laser appointment
- [x] another_project_name
```

### Visualization
- Progress bars for each project showing completion percentage using native Obsidian/Dataview progress bars
- Strikethrough for completed projects (100%)
- Weekly progress visualization with week number (e.g., "2/12") and date range
- Strikethrough for past weeks
- Preserve and display note links (e.g., [[Project Name]]) in all tables, maintaining clickable functionality

### Calculation Features
- Calculate total tasks across all projects
- Apply weighting to prioritize certain projects
- Calculate remaining tasks
- Determine weeks remaining in the date range
- Calculate required tasks per week to complete all projects

### Debug Mode
When enabled, debug mode displays:
- Total tasks across all projects
- Remaining tasks
- Weekly target
- Date calculations
- Project names

### Habit Tracking
- Track habits defined in the goals.md file
- Count completed habit tasks in daily notes
- Show weekly consistency for each habit
- Calculate consistency percentage based on weekly targets
- Display habit scorecard with visualization
- Show overall habit progress across all weeks with consistency average

#### Overall Habit Progress
The script will display an overall habit progress table with:
- One row per habit
- First column: Habit name
- Second column: Consistency average (average weekly count / weekly target with percentage)
- Third column: Progress bar visualization

#### Habits File Format (goals.md)
```markdown
- habit:: meditate
- weekly:: 4
- habit:: gym
- weekly:: 3
```

#### Daily Notes Format for Habits
Tasks in daily notes should exactly match the habit names:
```markdown
- [x] meditate
- [x] gym
```

## Potential Future Improvements
- Historical weekly progress tracking
- Burndown chart visualization
- Project categories or tags
- Project deadlines and priority levels
- More flexible task matching system
- Dashboard view with multiple visualizations
- Support for recurring projects
- Notifications for projects that are behind schedule
- Support for note links with aliases (e.g., [[Project Name|Alias]])
