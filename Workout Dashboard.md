---
tags:
  - fitness
  - workout-dashboard
cssclasses:
  - workout-dashboard
---
### Yearly activity heatmap

```dataviewjs
// YEARLY HEATMAP — Heatmap Calendar (Richardsl) + DataviewJS.
// Uses the plugin's documented renderHeatmapCalendar(this.container, calendarData).
const logFolder = [dv.current().file.folder, "Workout Logs"].filter(Boolean).join("/");
const completed = dv.pages(JSON.stringify(logFolder))
  .where(p => p.completed === true).array();
function dayOf(page) {
  if (page.date && typeof page.date.toISODate === "function") return page.date.toISODate();
  if (typeof page.date === "string") {
    const match = page.date.match(/^(\d{4}-\d{2}-\d{2})/);
    if (match) return match[1];
  }
  const match = page.file.name.match(/^(\d{4}-\d{2}-\d{2})/);
  return match ? match[1] : null;
}
const days = new Map();
for (const p of completed) {
  const date=dayOf(p);
  if (!date || !/^\d{4}-\d{2}-\d{2}$/.test(date)) continue;
  if (!days.has(date)) days.set(date, []);
  days.get(date).push(p);
}
function colorFor(session) {
  const s=String(session||"").toLowerCase();
  if (s.includes("cardio")) return "cardio";
  if (s.includes("chest")) return "chest";
  if (s.includes("back")) return "back";
  if (s.includes("shoulder") || s.includes("core") || s.includes("abs")) return "shoulders";
  if (s.includes("leg") || s.includes("calf")) return "legs";
  if (s.includes("recovery") || s.includes("mobility")) return "recovery";
  return "other";
}
const calendarData = {
  // Remove this line to use Heatmap Calendar's default current-year behavior.
  year: new Date().getFullYear(),
  showCurrentDayBorder: true,
  intensityScaleStart: 0,
  intensityScaleEnd: 120,
  colors: {
    chest: ["#f9d2d5","#ef9da4","#db7d85","#c95f68","#ac404c"],
    back: ["#cee5fa","#a5cbef","#7ab1e3","#4b88c9","#316daa"],
    shoulders: ["#e7d9f7","#d1b5ec","#b493dd","#956ac6","#784ca8"],
    legs: ["#d0f2e1","#a3dec1","#71c89d","#439a73","#287b56"],
    cardio: ["#fce9c9","#f3d394","#e6b862","#c79136","#a57426"],
    recovery: ["#e0e4e8","#c8d0d7","#acb9c4","#8996a4","#677787"],
    other: ["#d3ebed","#a6d3d5","#7bbabd","#489a9b","#2f797b"]
  },
  entries: []
};
for (const [date, logs] of days) {
  const duration=logs.reduce((sum,p)=>sum+(Number(p.duration_minutes)||0),0);
  calendarData.entries.push({
    date,
    intensity: duration>0?duration:30,
    content: logs.length>1?String(logs.length):"•",
    color: logs.length>1?"other":colorFor(logs[0].session)
  });
}
if (typeof renderHeatmapCalendar === "function") {
  const heatmapRoot = this.container.createDiv({cls:"wt-yearly"});
  heatmapRoot.createEl("style", {text: `
.workout-dashboard {--file-line-width:100%;}
.markdown-preview-view.workout-dashboard .markdown-preview-sizer,
.markdown-source-view.workout-dashboard .cm-sizer {width:100%;max-width:none !important;}
.wt-yearly {width:100%;min-width:0;overflow-x:auto;container-type:inline-size;padding:8px 0 10px;}
.wt-yearly .heatmap-calendar-graph {
  --wt-cell:clamp(12px,calc((100cqw - 162px) / 53),20px);
  display:grid;grid-template-columns:40px max-content;grid-template-rows:auto auto;
  column-gap:8px;row-gap:6px;width:max-content;min-width:100%;
  box-sizing:border-box;padding:5px;font-size:13px;line-height:1.2;
}
.wt-yearly .heatmap-calendar-graph ul {list-style:none !important;padding:0 !important;margin:0 !important;}
.wt-yearly .heatmap-calendar-graph li {padding:0;margin:0 !important;}
.wt-yearly .heatmap-calendar-months {display:grid !important;grid-template-columns:repeat(12,minmax(0,1fr));gap:0;}
.wt-yearly .heatmap-calendar-boxes {
  display:grid !important;grid-auto-flow:column;grid-template-columns:repeat(53,var(--wt-cell));
  grid-template-rows:repeat(7,var(--wt-cell));gap:2px;
}
.wt-yearly .heatmap-calendar-days {display:grid !important;grid-template-rows:repeat(7,var(--wt-cell));gap:2px;}
.wt-yearly .heatmap-calendar-days li {display:flex;align-items:center;font-size:11px;line-height:var(--wt-cell) !important;}
.wt-yearly .heatmap-calendar-boxes li {
  box-sizing:border-box;width:var(--wt-cell);height:var(--wt-cell);min-height:0;
  border-radius:2px;line-height:var(--wt-cell) !important;text-align:center;overflow:hidden;
}
.wt-yearly .heatmap-calendar-content {font-size:9px;line-height:inherit;}
`});
  renderHeatmapCalendar(heatmapRoot, calendarData);
} else {
  dv.paragraph("Heatmap Calendar is not enabled. Install **Heatmap Calendar by Richardsl**, then reopen this note in Reading view.");
}
```
%% col-start %%

%% col-break:50,b:primary %%

### Rotation

| Rotation day | Focus           |
| ------------ | --------------- |
| Mon          | Chest + Triceps |
| Tue          | Rest            |
| Wed          | Back + Biceps   |
| Thu          | Rest            |
| Fri          | Shoulders + Abs |
| Sat          | Rest            |
| Sun          | Legs            |

---
### Logs

```dataview
TABLE WITHOUT ID key AS "Month", length(rows) AS "Sessions", sum(rows.duration_minutes) AS "Minutes"
FROM ""
WHERE completed = true
AND startswith(file.path, choice(this.file.folder = "", "", this.file.folder + "/") + "Workout Logs/")
GROUP BY dateformat(date, "yyyy-MM") AS key
SORT key DESC
```

%% col-break:50,b:primary %%

### Monthly Calander

```dataviewjs
// MONTHLY CALENDAR + STATS — uses DataviewJS only.
// Reads completed workout notes; no special filename format required.
const logFolder = [dv.current().file.folder, "Workout Logs"].filter(Boolean).join("/");
const PAGES = dv.pages(JSON.stringify(logFolder))
  .where(p => p.completed === true).array();
const pad = n => String(n).padStart(2, "0");
const localDate = d => `${d.getFullYear()}-${pad(d.getMonth()+1)}-${pad(d.getDate())}`;
function getDay(page) {
  if (page.date && typeof page.date.toISODate === "function") return page.date.toISODate();
  if (typeof page.date === "string") {
    const match = page.date.match(/^(\d{4}-\d{2}-\d{2})/);
    if (match) return match[1];
  }
  const match = page.file.name.match(/^(\d{4}-\d{2}-\d{2})/);
  return match ? match[1] : null;
}
const workouts = PAGES.map(p => ({page:p, date:getDay(p)}))
  .filter(w => w.date && /^\d{4}-\d{2}-\d{2}$/.test(w.date))
  .sort((a,b) => b.date.localeCompare(a.date));
const byDate = new Map();
for (const w of workouts) {
  if (!byDate.has(w.date)) byDate.set(w.date, []);
  byDate.get(w.date).push(w.page);
}
const COLOR = { chest:"#c95f68", back:"#4b88c9", shoulder:"#956ac6", leg:"#439a73", cardio:"#c79136", recovery:"#8996a4", other:"#489a9b", multiple:"#489a9b" };
function category(session) {
  const s = String(session || "").toLowerCase();
  if (s.includes("cardio")) return "cardio";
  if (s.includes("chest")) return "chest";
  if (s.includes("back")) return "back";
  if (s.includes("shoulder") || s.includes("abs") || s.includes("core")) return "shoulder";
  if (s.includes("leg") || s.includes("calf")) return "leg";
  if (s.includes("rest") || s.includes("recovery") || s.includes("mobility")) return "recovery";
  return "other";
}
const today = new Date();
const nowKey = localDate(today);
const monday = new Date(today.getFullYear(), today.getMonth(), today.getDate() - (today.getDay()+6)%7);
const startOfWeek = localDate(monday);
const thisMonth = nowKey.slice(0,7);
const totalMinutes = workouts.reduce((sum,w)=>sum+(Number(w.page.duration_minutes)||0),0);

const root = this.container.createDiv({cls:"wt-dashboard"});
root.createEl("style", {text: `
.wt-dashboard {--wt-day-height:36px;margin:6px 0 12px;min-width:0;width:100%;max-width:420px;container-type:inline-size;}
.wt-dashboard .wt-stats {display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:4px;margin-bottom:20px !important;}
.wt-dashboard .wt-stat {display:flex;align-items:center;gap:5px;border:1px solid var(--background-modifier-border);background:var(--background-secondary);padding:4px 5px;border-radius:4px;min-width:0;}
.wt-dashboard .wt-number {font-size:15px;font-weight:700;line-height:1.1;color:var(--text-normal);flex-shrink:0;}
.wt-dashboard .wt-label {font-size:9px;line-height:1.2;color:var(--text-muted);margin:0;min-width:0;}
.wt-dashboard .wt-nav {display:grid;grid-template-columns:auto minmax(0,1fr) auto;align-items:center;gap:5px;margin:0 0 7px;}
.wt-dashboard .wt-month-title {font-size:14px;font-weight:700;text-align:center;}
.wt-dashboard .wt-nav-btn {height:26px;min-height:0;padding:0 6px;font-size:11px;border:1px solid var(--background-modifier-border);border-radius:5px;background:var(--interactive-normal);color:var(--text-normal);cursor:pointer;box-shadow:none;}
.wt-dashboard .wt-weekday,.wt-dashboard .wt-days {display:grid;grid-template-columns:repeat(7,minmax(0,1fr));gap:3px;}
.wt-dashboard .wt-weekday {margin-bottom:4px;text-align:center;font-size:10px;font-weight:600;color:var(--text-muted);}
.wt-dashboard .wt-days {grid-auto-rows:var(--wt-day-height);align-content:start;height:auto;min-height:0;}
.wt-dashboard .wt-days .wt-day {box-sizing:border-box;width:100%;height:var(--wt-day-height);min-width:0;min-height:0;max-height:var(--wt-day-height);margin:0;border-radius:5px;border:1px solid var(--background-modifier-border);background:var(--background-primary);color:var(--text-normal);text-align:left;padding:3px 4px;display:flex;flex-direction:column;align-items:stretch;justify-content:space-between;gap:1px;box-shadow:none;line-height:1.2;overflow:hidden;}
.wt-dashboard .wt-day:disabled {opacity:1;}
.wt-dashboard .wt-day.wt-outside {opacity:.35;}
.wt-dashboard .wt-day.wt-done {cursor:pointer;border-color:var(--wt-color);background:color-mix(in srgb,var(--wt-color) 18%,var(--background-primary));}
.wt-dashboard .wt-day.wt-done:hover {background:color-mix(in srgb,var(--wt-color) 30%,var(--background-primary));}
.wt-dashboard .wt-day.wt-today {outline:2px solid var(--interactive-accent);outline-offset:-2px;}
.wt-dashboard .wt-day-num {font-weight:650;font-size:12px;}
.wt-dashboard .wt-day-label {color:var(--text-normal);font-size:9px;line-height:1.2;overflow:hidden;text-overflow:ellipsis;white-space:nowrap;max-width:100%;}
.wt-dashboard .wt-legend {display:flex;gap:4px 7px;flex-wrap:wrap;margin-top:8px;font-size:10px;color:var(--text-muted);}
.wt-dashboard .wt-legend-item {display:inline-flex;align-items:center;gap:5px;}
.wt-dashboard .wt-swatch {width:9px;height:9px;border-radius:2px;display:inline-block;flex-shrink:0;}
.wt-dashboard .wt-help {font-size:11px;color:var(--text-muted);margin-top:9px;line-height:1.4;}
@container (max-width:320px){.wt-dashboard .wt-stats{grid-template-columns:repeat(2,minmax(0,1fr))}.wt-dashboard .wt-days{--wt-day-height:34px}.wt-dashboard .wt-nav-btn{padding:0 4px}.wt-dashboard .wt-month-title{font-size:12px}}
`});
const stats = root.createDiv({cls:"wt-stats"});
for (const [label, val] of [
  ["This week", workouts.filter(w=>w.date>=startOfWeek&&w.date<=nowKey).length],
  ["This month", workouts.filter(w=>w.date.startsWith(thisMonth)).length],
  ["All workouts", workouts.length],
  ["Total minutes", totalMinutes]
]) {
  const card = stats.createDiv({cls:"wt-stat"});
  card.createDiv({cls:"wt-number",text:String(val)});
  card.createDiv({cls:"wt-label",text:label});
}
const nav = root.createDiv({cls:"wt-nav"});
const previous = nav.createEl("button",{cls:"wt-nav-btn",text:"← Previous"});
const title = nav.createDiv({cls:"wt-month-title"});
const next = nav.createEl("button",{cls:"wt-nav-btn",text:"Next →"});
const header = root.createDiv({cls:"wt-weekday"});
for (const s of ["Mon","Tue","Wed","Thu","Fri","Sat","Sun"]) header.createDiv({text:s});
const calendar = root.createDiv({cls:"wt-days"});
let displayed = new Date(today.getFullYear(),today.getMonth(),1);
function renderMonth() {
  calendar.empty();
  const year=displayed.getFullYear(), month=displayed.getMonth();
  title.textContent=displayed.toLocaleDateString(undefined,{month:"long",year:"numeric"});
  const offset=(new Date(year,month,1).getDay()+6)%7;
  const count=Math.ceil((offset+new Date(year,month+1,0).getDate())/7)*7;
  for (let i=0;i<count;i++) {
    const dayDate=new Date(year,month,1+i-offset);
    const key=localDate(dayDate);
    const entries=byDate.get(key)||[];
    const active=entries.length>0;
    const cat=entries.length>1 ? "multiple" : category(entries[0]?.session);
    const tile=calendar.createEl("button",{cls:"wt-day"+(dayDate.getMonth()!==month?" wt-outside":"")+(active?" wt-done":"")+(key===nowKey?" wt-today":"")});
    tile.type="button";
    tile.disabled=!active;
    tile.setAttribute("aria-label",`${key}: ${active?entries.map(p=>p.session||"Workout").join(", "):"No completed workouts"}`);
    tile.createDiv({cls:"wt-day-num",text:String(dayDate.getDate())});
    if (active) {
      tile.style.setProperty("--wt-color",COLOR[cat]);
      const label=entries.length>1?`${entries.length} sessions`:String(entries[0].session||"Workout");
      tile.createDiv({cls:"wt-day-label",text:label});
      tile.title=entries.map(p=>`${p.session||"Workout"} · ${Number(p.duration_minutes)||0} min`).join("\n");
      tile.addEventListener("click",()=>app.workspace.openLinkText(entries[0].file.path,"",false));
    }
  }
}
previous.addEventListener("click",()=>{displayed=new Date(displayed.getFullYear(),displayed.getMonth()-1,1);renderMonth();});
next.addEventListener("click",()=>{displayed=new Date(displayed.getFullYear(),displayed.getMonth()+1,1);renderMonth();});
renderMonth();
const legend=root.createDiv({cls:"wt-legend"});
for (const [key,label] of [["chest","Chest"],["back","Back"],["shoulder","Shoulders / Core"],["leg","Legs"],["cardio","Cardio"],["recovery","Recovery"],["multiple","Other / multi"]]) {
  const item=legend.createDiv({cls:"wt-legend-item"});
  item.createEl("span",{cls:"wt-swatch"}).style.backgroundColor=COLOR[key];
  item.createSpan({text:label});
}
root.createDiv({cls:"wt-help"});
```

%% col-end %%

*"You have a right to perform your prescribed duties, but not to the fruits of your actions. 
Never consider yourself to be the cause of the results of your activities, and never be attached to not doing your duty." - Bhagavad Gita*