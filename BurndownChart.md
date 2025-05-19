# 12-Week Year Burndown Chart

```dataviewjs
// Get the goals page
const goalsPage = dv.page("goals");

// Extract date range and debug setting from goals.md
let startDateStr = "2024-01-01"; // Default start date
let endDateStr = "2024-12-31";   // Default end date
let debugMode = false;           // Default debug mode

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

// Extract projects from the list items
let projectsList = [];
let goalNames = []; // Store goal names for later use

if (goalsPage.file && goalsPage.file.lists) {
    // First pass: collect all projects and their properties
    for (let i = 0; i < goalsPage.file.lists.length; i++) {
        const item = goalsPage.file.lists[i];
        
        // If this is a project item, start a new project object
        if (item.project) {
            const cleaned = cleanContent(item.project);
            
            projectsList.push({
                name: cleaned.text,
                originalFormat: cleaned.originalFormat,
                isNoteLink: cleaned.isNoteLink,
                weight: 1,  // Default weight
                tasks: 0
            });
        }
        // If this is a weight item, update the last project's weight
        else if (item.weight !== undefined && projectsList.length > 0) {
            const weightValue = parseInt(String(item.weight).trim()) || 1;
            projectsList[projectsList.length - 1].weight = weightValue;
        }
        // If this is a tasks item, update the last project's tasks
        else if (item.tasks !== undefined && projectsList.length > 0) {
            const tasksValue = parseInt(String(item.tasks).trim()) || 0;
            projectsList[projectsList.length - 1].tasks = tasksValue;
        }
    }
    
    // Store project names for later use
    goalNames = projectsList.map(p => p.name).filter(name => name);
}

// Calculate total project effort (task*weight)
let totalProjectEffort = 0;
for (const project of projectsList) {
    if (project.name && project.tasks > 0) {
        totalProjectEffort += project.tasks * project.weight;
    }
}

// Function to get completed project effort (task*weight) for a specific date
function getCompletedProjectEffortForDate(date) {
    let completedProjectEffort = 0;
    
    // Find daily notes for this date
    const notesForDate = dailyNotes.filter(note => {
        try {
            const noteDate = new Date(note.file.name);
            return noteDate.toDateString() === date.toDateString();
        } catch (e) {
            return false;
        }
    });
    
    // Process each note for this date
    for (const note of notesForDate) {
        if (!note.file.tasks) continue;
        
        // Look for completed tasks that match our project names
        for (const task of note.file.tasks) {
            if (!task.completed) continue;
            
            // Remove checkbox markers and trim whitespace
            const taskText = task.text.replace(/^\s*\[x\]\s*/, '').trim();
            
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
            
            // Check if the task matches a project name
            for (const project of projectsList) {
                if (cleanedTaskText === project.name || cleanedTaskText.startsWith(project.name + " ")) {
                    completedProjectEffort += project.weight;
                    break;
                }
            }
        }
    }
    
    return completedProjectEffort;
}

// Generate daily data from start date to today
const dailyData = [];
let cumulativeCompletedEffort = 0;
let currentDate = new Date(startDate);

// Calculate days elapsed and total days
const msPerDay = 1000 * 60 * 60 * 24;
const daysElapsed = Math.max(1, Math.floor((today - startDate) / msPerDay));
const totalDays = Math.ceil((endDate - startDate) / msPerDay);

// Generate data for each day from start to today
while (currentDate <= today && currentDate <= endDate) {
    const dayNumber = Math.floor((currentDate - startDate) / msPerDay);
    const completedToday = getCompletedProjectEffortForDate(currentDate);
    cumulativeCompletedEffort += completedToday;
    
    dailyData.push({
        date: new Date(currentDate),
        day: dayNumber,
        completedToday: completedToday,
        cumulativeCompleted: cumulativeCompletedEffort
    });
    
    // Move to next day
    currentDate.setDate(currentDate.getDate() + 1);
}

// Calculate actual burn rate (daily and weekly)
const actualDailyBurnRate = daysElapsed > 0 ? cumulativeCompletedEffort / daysElapsed : 0;
const actualWeeklyBurnRate = actualDailyBurnRate * 7;

// Calculate remaining project effort
const remainingEffort = totalProjectEffort - cumulativeCompletedEffort;

// Calculate days remaining
const daysRemaining = Math.max(1, Math.ceil((endDate - today) / msPerDay));

// Calculate target burn rate to meet deadline
const targetDailyBurnRate = remainingEffort / daysRemaining;
const targetWeeklyBurnRate = targetDailyBurnRate * 7;

// Calculate target burn rate for 80% completion
const effortFor80Percent = totalProjectEffort * 0.8;
const remainingEffortFor80Percent = Math.max(0, effortFor80Percent - cumulativeCompletedEffort);
const targetDailyBurnRateFor80Percent = remainingEffortFor80Percent / daysRemaining;
const targetWeeklyBurnRateFor80Percent = targetDailyBurnRateFor80Percent * 7;

// Display burn rate information
dv.header(2, "Burndown Rates");

// Format numbers to 2 decimal places
function formatNumber(num) {
    return Math.round(num * 100) / 100;
}

// Create a table for burn rates
const burnRateTable = [
    ["Actual Daily Burn Rate", formatNumber(actualDailyBurnRate)],
    ["Actual Weekly Burn Rate", formatNumber(actualWeeklyBurnRate)],
    ["Target Daily Burn Rate (100%)", formatNumber(targetDailyBurnRate)],
    ["Target Weekly Burn Rate (100%)", formatNumber(targetWeeklyBurnRate)],
    ["Target Daily Burn Rate (80%)", formatNumber(targetDailyBurnRateFor80Percent)],
    ["Target Weekly Burn Rate (80%)", formatNumber(targetWeeklyBurnRateFor80Percent)]
];

dv.table(["Metric", "Value"], burnRateTable);

// Calculate completion date projections
const projectedCompletionDate = new Date(today);
if (actualDailyBurnRate > 0) {
    const daysToCompletion = remainingEffort / actualDailyBurnRate;
    projectedCompletionDate.setDate(projectedCompletionDate.getDate() + Math.ceil(daysToCompletion));
}

const projectedCompletion80Date = new Date(today);
if (actualDailyBurnRate > 0) {
    const daysTo80Completion = remainingEffortFor80Percent / actualDailyBurnRate;
    projectedCompletion80Date.setDate(projectedCompletion80Date.getDate() + Math.ceil(daysTo80Completion));
}

// Format dates
const dateFormatter = new Intl.DateTimeFormat('en-US', { 
    year: 'numeric', 
    month: 'short', 
    day: 'numeric' 
});

// Display projections
dv.header(2, "Completion Projections");
const projectionTable = [
    ["End Date (Deadline)", dateFormatter.format(endDate)],
    ["Projected 100% Completion Date (actual burn rate)", dateFormatter.format(projectedCompletionDate)],
    ["Projected 80% Completion Date (actual burn rate)", dateFormatter.format(projectedCompletion80Date)]
];

dv.table(["Milestone", "Date"], projectionTable);

// Calculate if on track
const onTrack = projectedCompletionDate <= endDate;
const on80Track = projectedCompletion80Date <= endDate;

// Display status
dv.paragraph(`**Status:** ${onTrack ? "✅ On track to complete 100% by deadline" : "⚠️ Not on track to complete 100% by deadline"}`);
dv.paragraph(`**80% Status:** ${on80Track ? "✅ On track to complete 80% by deadline" : "⚠️ Not on track to complete 80% by deadline"}`);

// Display daily effort completion table
// Debug information if enabled
if (debugMode) {
    dv.header(3, "Debug Information");
    dv.paragraph(`**Start Date:** ${startDate.toDateString()}`);
    dv.paragraph(`**End Date:** ${endDate.toDateString()}`);
    dv.paragraph(`**Today:** ${today.toDateString()}`);
    dv.paragraph(`**Projects:** ${JSON.stringify(projectsList)}`);
    dv.paragraph(`**Daily Data:** ${JSON.stringify(dailyData)}`);
}

```
