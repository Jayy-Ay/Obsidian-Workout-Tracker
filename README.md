# Obsidian Workout Dashboard

[![Obsidian](https://img.shields.io/badge/Obsidian-7C3AED?style=flat-square&logo=obsidian&logoColor=white)](https://obsidian.md/)
[![Dataview](https://img.shields.io/badge/Plugin-Dataview-2563EB?style=flat-square)](https://github.com/blacksmithgu/obsidian-dataview)
[![Heatmap Calendar](https://img.shields.io/badge/Plugin-Heatmap%20Calendar-D97706?style=flat-square)](https://github.com/Richardsl/heatmap-calendar-obsidian)
[![Advanced Multi Column](https://img.shields.io/badge/Plugin-Advanced%20Multi%20Column-7C3AED?style=flat-square)](https://github.com/amatya-aditya/advanced-multi-column)

A workout tracker built from Markdown notes for Obsidian. It includes a yearly activity heatmap, a monthly calendar with workout statistics, an editable training rotation, and monthly session and minute totals.

## Files

- `Workout Dashboard.md` - the dashboard, including its queries and styling.
- `Workout Log Template.md` - a template for recording a workout.
- `Workout Logs/` - dated workout notes read by the dashboard.

The repository stores the notes and their source code. Open the dashboard in Obsidian to use the interactive calendars.

## Setup

1. Download this repository as a ZIP or clone it, then copy the folder into an Obsidian vault. You can rename the folder; keep the dashboard and `Workout Logs/` together.
2. Install and enable these Obsidian community plugins:
   - [Dataview](https://github.com/blacksmithgu/obsidian-dataview)
   - [Heatmap Calendar by Richardsl](https://github.com/Richardsl/heatmap-calendar-obsidian)
   - [Advanced Multi Column](https://github.com/amatya-aditya/advanced-multi-column)
3. In **Settings → Dataview**, enable **JavaScript Queries**.
4. Open `Workout Dashboard.md` in **Reading view** or **Live Preview**.
5. To insert workout templates, enable Obsidian's built-in **Templates** plugin and select this folder as its template folder.

## Record a workout

1. Create a note inside `Workout Logs/`, named `YYYY-MM-DD - Workout`, such as `2026-10-11 - Legs`.
2. Insert `Workout Log Template` and enter the exercises, sets, repetitions, and any notes.
3. Set `date` to the workout date and `duration_minutes` to the time spent training, if recorded.
4. Set `completed: true` after finishing. The calendar and monthly totals use completed notes only.

An optional `session` property, such as `session: "Legs"` or `session: "Chest + Triceps"`, controls the calendar's workout category and colour. Notes without it use the general workout colour. Multiple sessions on one date are counted separately, and clicking the date opens its first workout note.
