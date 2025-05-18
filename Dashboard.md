```dataviewjs
// Get the goals page
const goalsPage = dv.page("goals");

// Extract date range and debug setting from goals.md
let startDateStr = "2024-01-01"; // Default start date
let endDateStr = "2024-12-31";   // Default end date
let debugMode = false;           // Default debug mode

// Store habits and their weekly targets
let habits = [];

// Helper function to extract clean content from Dataview objects or strings
function cleanContent(value) {
    if (!value) return null;
    
    // Convert to string
    let strValue = String(value);
    let cleanedText = "";
    let isNoteLink = false;
    
    // Check if this is a note link format like [[2025-05-12]]
    if (strValue.includes("[[") && strValue.includes("]]")) {
        isNoteLink = true;
        
        // Extract content inside brackets for comparison purposes
        const linkMatch = strValue.match(/\[\[(.*?)\]\]/);
        if (linkMatch) {
            const linkContent = linkMatch[1];
            
            // Handle potential pipe notation inside the link
            if (linkContent.includes("|")) {
                cleanedText = linkContent.split("|").pop().trim();
            } else {
                cleanedText = linkContent.trim();
            }
        } else {
            cleanedText = strValue.replace(/\[\[/g, "").replace(/\]\]/g, "").trim();
        }
    } else {
        // If not a note link, just clean it for comparison
        cleanedText = strValue.trim();
    }
    
    // Remove file extension if present (for dates that might be note names)
    if (cleanedText.endsWith(".md")) {
        cleanedText = cleanedText.slice(0, -3);
    }
    
    return {
        text: cleanedText,
        originalFormat: strValue,
        isNoteLink: isNoteLink
    };
}

// Look for Start, End, and Debug in the lists
if (goalsPage.file && goalsPage.file.lists) {
    for (const item of goalsPage.file.lists) {
        if (item.Start) {
            const cleaned = cleanContent(item.Start);
            if (cleaned.text) {
                startDateStr = cleaned.text;
            }
        }
        if (item.End) {
            const cleaned = cleanContent(item.End);
            if (cleaned.text) {
                endDateStr = cleaned.text;
            }
        }
        if (item.Debug !== undefined) {
            debugMode = String(item.Debug).toLowerCase() === "true";
        }
    }
}

// Parse dates
const startDate = new Date(startDateStr);
const endDate = new Date(endDateStr);
const today = new Date();

// Get all daily notes in the date range
const dailyNotes = dv.pages('"Daily note"').where(p => {
    try {
        const noteDate = new Date(p.file.name);
        return noteDate >= startDate && noteDate <= endDate;
    } catch (e) {
        return false;
    }
});

// Custom progress bar HTML generator
function createProgressBar(value, max, isComplete) {
    // Calculate percentage
    const percentage = max > 0 ? Math.round((value / max) * 100) : 0;
    const percentageText = `${value}/${max} (${percentage}%)`;
    
    // For completed items, use a custom HTML structure with green color
    if (isComplete) {
        // Create a div with green background to represent a completed progress bar
        return `${percentageText} <div style="display: inline-block; width: 200px; height: 10px; background-color: #4CAF50; border-radius: 5px;"></div>`;
    } else {
        // Use the default progress element for incomplete items
        return `${percentageText} <progress value="${value}" max="${max}"></progress>`;
    }
}

// Extract goals from the list items
if (goalsPage.file && goalsPage.file.lists) {
    // Collect all goals and their data
    let allGoals = [];
    let totalStepsAllGoals = 0;
    let totalCompletedStepsAllGoals = 0;
    let goalNames = []; // Store goal names for later use
    
    // Debug the lists structure if debug mode is enabled
    if (debugMode) {
        dv.paragraph("**Debug - Lists Structure:**");
        for (let i = 0; i < goalsPage.file.lists.length; i++) {
            const item = goalsPage.file.lists[i];
            dv.paragraph(`List item ${i}: ${JSON.stringify(item)}`);
        }
    }
    
    // Extract all projects first
    let projectsList = [];
    
    // First pass: collect all projects, habits and their properties
    for (let i = 0; i < goalsPage.file.lists.length; i++) {
        const item = goalsPage.file.lists[i];
        
        // If this is a project item, start a new project object
        if (item.project) {
            const cleaned = cleanContent(item.project);
            
            // Debug the project item
            if (debugMode) {
                dv.paragraph(`Project item: ${JSON.stringify(item.project)}`);
                dv.paragraph(`Cleaned project: ${JSON.stringify(cleaned)}`);
            }
            
            projectsList.push({
                name: cleaned.text,
                originalFormat: cleaned.originalFormat,
                isNoteLink: cleaned.isNoteLink,
                weight: 1,  // Default weight
                tasks: 0
            });
        }
        // If this is a habit item, start a new habit object
        else if (item.habit) {
            const cleaned = cleanContent(item.habit);
            
            // Debug the habit item
            if (debugMode) {
                dv.paragraph(`Habit item: ${JSON.stringify(item.habit)}`);
                dv.paragraph(`Cleaned habit: ${JSON.stringify(cleaned)}`);
            }
            
            habits.push({
                name: cleaned.text,
                originalFormat: cleaned.originalFormat,
                isNoteLink: cleaned.isNoteLink,
                weekly: 0  // Default weekly target
            });
        }
        // If this is a weight item, update the last project's weight
        else if (item.weight !== undefined && projectsList.length > 0) {
            const weightValue = parseInt(String(item.weight).trim()) || 1;
            projectsList[projectsList.length - 1].weight = weightValue;
            
            if (debugMode) {
                dv.paragraph(`Setting weight for ${projectsList[projectsList.length - 1].name}: ${weightValue}`);
            }
        }
        // If this is a weekly item, update the last habit's weekly target
        else if (item.weekly !== undefined && habits.length > 0) {
            const weeklyValue = parseInt(String(item.weekly).trim()) || 0;
            habits[habits.length - 1].weekly = weeklyValue;
            
            if (debugMode) {
                dv.paragraph(`Setting weekly target for ${habits[habits.length - 1].name}: ${weeklyValue}`);
            }
        }
        // If this is a tasks item, update the last project's tasks
        else if (item.tasks !== undefined && projectsList.length > 0) {
            const tasksValue = parseInt(String(item.tasks).trim()) || 0;
            projectsList[projectsList.length - 1].tasks = tasksValue;
            
            if (debugMode) {
                dv.paragraph(`Setting tasks for ${projectsList[projectsList.length - 1].name}: ${tasksValue}`);
            }
        }
    }
    
    // Debug the collected habits
    if (debugMode) {
        dv.paragraph("**Debug - Collected Habits:**");
        for (const habit of habits) {
            dv.paragraph(`Habit: ${habit.name}, Weekly Target: ${habit.weekly}`);
        }
    }
    
    // Debug the collected projects
    if (debugMode) {
        dv.paragraph("**Debug - Collected Projects:**");
        for (const project of projectsList) {
            dv.paragraph(`Project: ${project.name}, Weight: ${project.weight}, Tasks: ${project.tasks}`);
        }
    }
    
    // Process each project
    for (const project of projectsList) {
        if (!project.name || project.tasks <= 0) continue;
        
        const weightedTasks = project.tasks * project.weight;
        totalStepsAllGoals += weightedTasks;
        
        // Store project name for later use
        goalNames.push(project.name);
        
        // Count completed tasks for this project
        let completedCount = 0;
        
        for (const note of dailyNotes) {
            if (!note.file.tasks) continue;
            
            // Look for exact matches to the project name (handling note links in tasks)
            const completedTasks = note.file.tasks.filter(t => {
                // Check if the task is completed
                if (!t.completed) return false;
                
                // Remove checkbox markers and trim whitespace
                const taskText = t.text.replace(/^\s*\[x\]\s*/, '').trim();
                
                // Handle note links in task text
                let cleanedTaskText = taskText;
                if (taskText.includes("[[") && taskText.includes("]]")) {
                    const linkMatch = taskText.match(/\[\[(.*?)\]\]/);
                    if (linkMatch) {
                        const linkContent = linkMatch[1];
                        // Handle potential pipe notation inside the link
                        if (linkContent.includes("|")) {
                            cleanedTaskText = linkContent.split("|").pop().trim();
                        } else {
                            cleanedTaskText = linkContent.trim();
                        }
                    }
                }
                
                return cleanedTaskText === project.name;
            });
            
            completedCount += completedTasks.length;
        }
        
        // Apply weight to completed count
        const weightedCompletedCount = completedCount * project.weight;
        totalCompletedStepsAllGoals += weightedCompletedCount;
        
        // Calculate percentage (handle division by zero)
        let percentage = 0;
        if (weightedTasks > 0) {
            percentage = Math.round((weightedCompletedCount / weightedTasks) * 100);
        }
        
        // Create progress bar HTML
        const isComplete = percentage >= 100;
        const progressBar = createProgressBar(weightedCompletedCount, weightedTasks, isComplete);
        
        // Format project name with strikethrough if 100% complete
        // Always preserve the original format (with or without brackets)
        let displayName = project.originalFormat;
        if (isComplete) {
            displayName = `<s>${project.originalFormat}</s>`;
        }
        
        // Add to all goals array
        allGoals.push([displayName, `${project.tasks} (x${project.weight})`, progressBar]);
    }
    
    // Display all projects in a single table
    dv.header(2, "Project Progress");
    dv.table(["Project", "Tasks", "Progress"], allGoals);
    
    // Weekly progress calculation
    dv.header(2, "Weekly Progress");
    
    // Calculate remaining tasks
    const remainingTasks = totalStepsAllGoals - totalCompletedStepsAllGoals;
    
    // Calculate number of weeks remaining
    const msPerWeek = 1000 * 60 * 60 * 24 * 7;
    const weeksRemaining = Math.max(1, Math.ceil((endDate - today) / msPerWeek));
    
    // Calculate tasks per week needed
    const tasksPerWeek = Math.ceil(remainingTasks / weeksRemaining);
    
    // Generate all weeks in the epoch
    const allWeeks = [];
    let currentWeekStart = new Date(startDate);
    
    // Calculate total number of weeks in the plan
    const totalWeeks = Math.ceil((endDate - startDate) / msPerWeek);
    let weekCounter = 1;
    
    // Adjust to start on Monday
    const startDayOfWeek = currentWeekStart.getDay();
    const daysToMonday = startDayOfWeek === 0 ? 1 : (startDayOfWeek === 1 ? 0 : 8 - startDayOfWeek);
    currentWeekStart.setDate(currentWeekStart.getDate() + daysToMonday);
    currentWeekStart.setHours(0, 0, 0, 0);
    
    // Format for week dates
    const weekDateFormat = new Intl.DateTimeFormat('en-US', { month: 'short', day: 'numeric' });
    
    // Generate weeks up to today (don't show future weeks)
    while (currentWeekStart <= endDate && currentWeekStart <= today) {
        const weekEnd = new Date(currentWeekStart);
        weekEnd.setDate(currentWeekStart.getDate() + 6);
        weekEnd.setHours(23, 59, 59, 999);
        
        // All weeks shown are now in the past or current
        const isPastWeek = weekEnd < today;
        
        // Count completed steps for this week
        let completedStepsThisWeek = 0;
        
        for (const note of dailyNotes) {
            try {
                const noteDate = new Date(note.file.name);
                
                // Check if note is from this week
                if (noteDate >= currentWeekStart && noteDate <= weekEnd) {
                    if (note.file.tasks) {
                        // Only count completed tasks that match our project names (handling note links)
                        const completedTasks = note.file.tasks.filter(t => {
                            if (!t.completed) return false;
                            
                            const taskText = t.text.replace(/^\s*\[x\]\s*/, '').trim();
                            
                            // Handle note links in task text
                            let cleanedTaskText = taskText;
                            if (taskText.includes("[[") && taskText.includes("]]")) {
                                const linkMatch = taskText.match(/\[\[(.*?)\]\]/);
                                if (linkMatch) {
                                    const linkContent = linkMatch[1];
                                    // Handle potential pipe notation inside the link
                                    if (linkContent.includes("|")) {
                                        cleanedTaskText = linkContent.split("|").pop().trim();
                                    } else {
                                        cleanedTaskText = linkContent.trim();
                                    }
                                }
                            }
                            
                            return goalNames.includes(cleanedTaskText);
                        });
                        
                        completedStepsThisWeek += completedTasks.length;
                    }
                }
            } catch (e) {
                // Skip notes with invalid dates
            }
        }
        
        // Calculate week score based on weekly target
        let weekScorePercentage = 0;
        if (tasksPerWeek > 0) {
            weekScorePercentage = Math.min(100, Math.round((completedStepsThisWeek / tasksPerWeek) * 100));
        }
        
        // Create week progress bar
        const isComplete = weekScorePercentage >= 100;
        const weekProgressBar = createProgressBar(completedStepsThisWeek, tasksPerWeek, isComplete);
        
        // Format week label with strikethrough if it's a past week
        const weekLabel = `${weekDateFormat.format(currentWeekStart)} - ${weekDateFormat.format(weekEnd)}`;
        const formattedWeekLabel = isPastWeek ? `<s>${weekLabel}</s>` : weekLabel;
        
        // Add week index column
        const weekIndex = `${weekCounter}/${totalWeeks}`;
        
        // Add to weeks array
        allWeeks.push([weekIndex, formattedWeekLabel, weekProgressBar]);
        
        // Increment week counter
        weekCounter++;
        
        // Move to next week
        currentWeekStart.setDate(currentWeekStart.getDate() + 7);
    }
    
    // Display debug information if enabled
    if (debugMode) {
        dv.paragraph(`**Total Tasks Across All Projects:** ${totalStepsAllGoals}`);
        dv.paragraph(`**Remaining Tasks:** ${remainingTasks}`);
        dv.paragraph(`**Weekly Target:** ${tasksPerWeek} tasks per week (${weeksRemaining} weeks remaining)`);
        
        // Debug date calculations
        dv.paragraph(`**Debug - Today:** ${today.toDateString()}`);
        dv.paragraph(`**Debug - Start Date:** ${startDate.toDateString()} (from: ${startDateStr})`);
        dv.paragraph(`**Debug - End Date:** ${endDate.toDateString()} (from: ${endDateStr})`);
        dv.paragraph(`**Debug - Project Names:** ${goalNames.join(", ")}`);
    }
    
    // Display weekly progress with all weeks
    dv.table(
        ["Week", "Days", "Progress"],
        allWeeks
    );
    
    // Process habits and create habit scorecard
    if (habits.length > 0) {
        dv.header(2, "Habit Scorecard");
        
        // Store habit names for later use
        let habitNames = habits.map(h => h.name);
        
        // Reset the week counter and currentWeekStart for habit tracking
        weekCounter = 1;
        currentWeekStart = new Date(startDate);
        
        // Adjust to start on Monday again
        currentWeekStart.setDate(currentWeekStart.getDate() + daysToMonday);
        currentWeekStart.setHours(0, 0, 0, 0);
        
        // Create array to store habit scorecard data
        const habitScorecard = [];
        
        // Generate weeks up to today (don't show future weeks)
        while (currentWeekStart <= endDate && currentWeekStart <= today) {
            const weekEnd = new Date(currentWeekStart);
            weekEnd.setDate(currentWeekStart.getDate() + 6);
            weekEnd.setHours(23, 59, 59, 999);
            
            // All weeks shown are now in the past or current
            const isPastWeek = weekEnd < today;
            
            // Format week label with strikethrough if it's a past week
            const weekLabel = `${weekDateFormat.format(currentWeekStart)} - ${weekDateFormat.format(weekEnd)}`;
            const formattedWeekLabel = isPastWeek ? `<s>${weekLabel}</s>` : weekLabel;
            
            // Add week index column
            const weekIndex = `${weekCounter}/${totalWeeks}`;
            
            // Process each habit for this week
            for (const habit of habits) {
                if (!habit.name || habit.weekly <= 0) continue;
                
                // Count completed tasks for this habit in this week
                let completedCount = 0;
                
                for (const note of dailyNotes) {
                    try {
                        const noteDate = new Date(note.file.name);
                        
                        // Check if note is from this week
                        if (noteDate >= currentWeekStart && noteDate <= weekEnd) {
                            if (note.file.tasks) {
                                // Look for exact matches to the habit name (handling note links)
                                const completedTasks = note.file.tasks.filter(t => {
                                    // Check if the task is completed
                                    if (!t.completed) return false;
                                    
                                    // Remove checkbox markers and trim whitespace
                                    const taskText = t.text.replace(/^\s*\[x\]\s*/, '').trim();
                                    
                                    // Handle note links in task text
                                    let cleanedTaskText = taskText;
                                    if (taskText.includes("[[") && taskText.includes("]]")) {
                                        const linkMatch = taskText.match(/\[\[(.*?)\]\]/);
                                        if (linkMatch) {
                                            const linkContent = linkMatch[1];
                                            // Handle potential pipe notation inside the link
                                            if (linkContent.includes("|")) {
                                                cleanedTaskText = linkContent.split("|").pop().trim();
                                            } else {
                                                cleanedTaskText = linkContent.trim();
                                            }
                                        }
                                    }
                                    
                                    return cleanedTaskText === habit.name;
                                });
                                
                                completedCount += completedTasks.length;
                            }
                        }
                    } catch (e) {
                        // Skip notes with invalid dates
                    }
                }
                
                // Calculate consistency percentage
                let consistencyPercentage = 0;
                if (habit.weekly > 0) {
                    consistencyPercentage = Math.min(100, Math.round((completedCount / habit.weekly) * 100));
                }
                
                // Create progress bar HTML
                const isComplete = consistencyPercentage >= 100;
                const progressBar = createProgressBar(completedCount, habit.weekly, isComplete);
                
                // Add to habit scorecard array
                habitScorecard.push([
                    weekIndex, 
                    formattedWeekLabel, 
                    habit.originalFormat,
                    progressBar
                ]);
            }
            
            // Increment week counter
            weekCounter++;
            
            // Move to next week
            currentWeekStart.setDate(currentWeekStart.getDate() + 7);
        }
        
        // Display habit scorecard
        dv.table(
            ["Week", "Days", "Habit", "Consistency"],
            habitScorecard
        );
        
        // Calculate overall habit progress across all weeks
        dv.header(2, "Overall Habit Progress");
        
        // Create array to store overall habit progress data
        const overallHabitProgress = [];
        
        // Process each habit for overall progress
        for (const habit of habits) {
            if (!habit.name || habit.weekly <= 0) continue;
            
            // Variables to track total counts and weeks
            let totalCompletedCount = 0;
            let totalWeeklyTarget = 0;
            let weeksTracked = 0;
            
            // Reset week counter and currentWeekStart
            let weekStartDate = new Date(startDate);
            weekStartDate.setDate(weekStartDate.getDate() + daysToMonday);
            weekStartDate.setHours(0, 0, 0, 0);
            
            // Process each week up to today
            while (weekStartDate <= endDate && weekStartDate <= today) {
                const weekEndDate = new Date(weekStartDate);
                weekEndDate.setDate(weekStartDate.getDate() + 6);
                weekEndDate.setHours(23, 59, 59, 999);
                
                // Count completed tasks for this habit in this week
                let weeklyCompletedCount = 0;
                
                for (const note of dailyNotes) {
                    try {
                        const noteDate = new Date(note.file.name);
                        
                        // Check if note is from this week
                        if (noteDate >= weekStartDate && noteDate <= weekEndDate) {
                            if (note.file.tasks) {
                                // Look for exact matches to the habit name (handling note links)
                                const completedTasks = note.file.tasks.filter(t => {
                                    // Check if the task is completed
                                    if (!t.completed) return false;
                                    
                                    // Remove checkbox markers and trim whitespace
                                    const taskText = t.text.replace(/^\s*\[x\]\s*/, '').trim();
                                    
                                    // Handle note links in task text
                                    let cleanedTaskText = taskText;
                                    if (taskText.includes("[[") && taskText.includes("]]")) {
                                        const linkMatch = taskText.match(/\[\[(.*?)\]\]/);
                                        if (linkMatch) {
                                            const linkContent = linkMatch[1];
                                            // Handle potential pipe notation inside the link
                                            if (linkContent.includes("|")) {
                                                cleanedTaskText = linkContent.split("|").pop().trim();
                                            } else {
                                                cleanedTaskText = linkContent.trim();
                                            }
                                        }
                                    }
                                    
                                    return cleanedTaskText === habit.name;
                                });
                                
                                weeklyCompletedCount += completedTasks.length;
                            }
                        }
                    } catch (e) {
                        // Skip notes with invalid dates
                    }
                }
                
                // Add to totals
                totalCompletedCount += weeklyCompletedCount;
                totalWeeklyTarget += habit.weekly;
                weeksTracked++;
                
                // Move to next week
                weekStartDate.setDate(weekStartDate.getDate() + 7);
            }
            
            // Calculate average consistency
            let averageCount = 0;
            let consistencyPercentage = 0;
            
            if (weeksTracked > 0) {
                averageCount = totalCompletedCount / weeksTracked;
                consistencyPercentage = Math.round((totalCompletedCount / totalWeeklyTarget) * 100);
            }
            
            // Create progress bar HTML
            const isComplete = consistencyPercentage >= 100;
            const progressBar = createProgressBar(totalCompletedCount, totalWeeklyTarget, isComplete);
            
            // Add to overall habit progress array
            overallHabitProgress.push([
                habit.originalFormat,
                progressBar
            ]);
        }
        
        // Display overall habit progress
        dv.table(
            ["Habit", "Consistency"],
            overallHabitProgress
        );
        
        // Debug habit information if enabled
        if (debugMode) {
            dv.paragraph("**Debug - Habit Information:**");
            for (const habit of habits) {
                dv.paragraph(`Habit: ${habit.name}, Weekly Target: ${habit.weekly}`);
            }
        }
    }
    
} else {
    dv.paragraph("No goals found in the expected format.");
}

```