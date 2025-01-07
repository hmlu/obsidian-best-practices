---
<%*
let url = 'https://www.tianqi.com/shanghai/';
let res = await request({ url: url, method: "GET" });

res = res.replace(/\s/g, ''); 

let r = /<ddclass="weather">[\s\S]*?<\/dd>/g;
let data = r.exec(res)[0];

r = /<span><b>(.*?)<\/b>(.*?)<\/span>/g;
data = r.exec(data);

let weather = '上海' + ' ' + data[1] + ' ' + data[2];
-%>
Type: Daily
Date: <% tp.file.creation_date("YYYY-MM-DD") %>
Weather: <% weather %>
tags: []
LearningEnglish:
Reading:
---
## 🚀GET READY

### ⭐ Top Tasks

### 🎯 Daily Focus

### ⏳ Pending Tasks Within 30 Days
```dataviewjs
const currentFile = dv.current();
const currentFileName = currentFile.file.name;
const currentFileCreationDate = new Date(currentFile.file.cday);
const fourteenDaysAgo = new Date(currentFileCreationDate);
fourteenDaysAgo.setDate(currentFileCreationDate.getDate() - 30);

// 获取所有任务
const allTasks = dv.pages('"日记"')
    .where(p => p.file && p.file.cday && p.file.tasks) // 确保文件、创建日期和任务存在
    .flatMap(p => p.file.tasks
        .filter(t => t) // 确保任务存在
        .filter(t => p.file.name !== currentFileName) // 排除当前笔记中的任务
        .filter(t => new Date(p.file.cday) >= fourteenDaysAgo) // 过滤创建日期在过去 14 天内的任务
        .filter(t => new Date(p.file.cday) < currentFileCreationDate)
        .map(t => ({
            ...t,
            text: `${t.text}` // 将笔记创建日期添加到任务文本中
        }))
    );

let currentTasks = dv.current().file.tasks;
let currentCompletedTasks = currentTasks.filter(t => t.completed);
let currentIncompleteTasks = currentTasks.filter(t => !t.completed);

// 计算总任务数和已完成任务数
const totalTasks = allTasks.length;
const completedTasks = allTasks.filter(t => t.completed);
const incompleteTasks = allTasks.filter(t => !t.completed);
const progressPercentage = ((completedTasks.length + currentCompletedTasks.length) / (totalTasks + currentTasks.length)) * 100;

// 显示任务列表
dv.taskList(incompleteTasks);

// 显示任务完成进度条
dv.paragraph(`**Task Completion Rate** <span style="color: green;">${progressPercentage.toFixed(0)}%</span> ( <span style="color: gray;">Total: <span style="color: orange;">${totalTasks + currentTasks.length}</span> Completed: <span style="color: green;">${completedTasks.length + currentCompletedTasks.length}</span> Incomplete: <span style="color: red;">${incompleteTasks.length + currentIncompleteTasks.length}</span></span> )`);
dv.paragraph(`
<div style="width: 100%; margin-top: -10px; background-color: #d3d3d3; border-radius: 3px; height: 3px; overflow: hidden;">
  <div style="width: ${progressPercentage}%; height: 3px; background-color: #4caf50;"></div>
</div>
`);
```
### 🤝 Team Coordination

#### **📝 Team Meetings**
```dataviewjs
let currentFile = dv.current().file;
let cday = currentFile.cday.toFormat("yyyy-MM-dd");

let sameDayMeetings = dv.pages('"工作笔记/工作记录/会议记录"')
    .where(p => p.file.cday.toFormat("yyyy-MM-dd") == cday)
    .where(p => p.Type && p.Type == "Meeting")
    .where(p => p.MeetingType && p.MeetingType == "Internal");

sameDayMeetings = sameDayMeetings.sort(p => p.file.ctime, 'asc');

if (sameDayMeetings.length > 0){
	dv.table(["File", "Created Time", "Summary"], 
    sameDayMeetings.map(p => [p.file.link, p.file.ctime.toFormat("HH:mm:ss"), p.summary]));
}
```
#### **🌍 External Meetings**
```dataviewjs
let currentFile = dv.current().file;
let cday = currentFile.cday.toFormat("yyyy-MM-dd");

let sameDayMeetings = dv.pages('"工作笔记/工作记录/会议记录"')
    .where(p => p.file.cday.toFormat("yyyy-MM-dd") == cday)
    .where(p => p.Type && p.Type == "Meeting")
    .where(p => p.MeetingType && p.MeetingType == "Client");

sameDayMeetings = sameDayMeetings.sort(p => p.file.ctime, 'asc');

if (sameDayMeetings.length > 0){
	dv.table(["File", "Created Time", "Summary"], 
    sameDayMeetings.map(p => [p.file.link, p.file.ctime.toFormat("HH:mm:ss"), p.summary]));
}
```
#### **✅ Meeting Tasks**
```dataviewjs
 let currentFile = dv.current().file;
let cday = currentFile.cday.toFormat("yyyy-MM-dd");

let sameDayMeetings = dv.pages('"工作笔记/工作记录/会议记录"')
    .where(p => p.file.cday.toFormat("yyyy-MM-dd") == cday)
    .where(p => p.Type && p.Type == "Meeting");

sameDayMeetings = sameDayMeetings.sort(p => p.file.ctime, 'asc');

if (sameDayMeetings.length > 0) {
    // 获取同一天所有会议笔记中的所有任务
    let allTasks = sameDayMeetings.flatMap(p => p.file.tasks);

    // 显示所有任务
    if (allTasks.length > 0) {
        dv.taskList(allTasks, { 
            filter: t => true,  // 可在这里进一步筛选任务
            map: t => {
                return {
                    text: `${t.text} (${t.completed ? "Completed" : "Incomplete"})`,
                    completed: t.completed
                };
            }
        });
    } else {
    }
} else {
}
```
#### **🛠️ Issues Resolved**

#### **📢 Support Needed**

---

## 🔍 REVIEW

### 📅 What Happend Today

### ✨ About Today

---

## 🗒️ Quick Notes

---

## ⏱️ Time Tracking
```dataviewjs
const tasks = dv.current().file.tasks;
const tagTimeMap = {};
let totalTime = 0;

// 调试信息
console.log("Tasks:", tasks);

// 将分钟转换为更易读的格式
const formatTime = (minutes) => {
    if (minutes >= 60) {
        const hours = Math.floor(minutes / 60);
        const mins = minutes % 60;
        return mins > 0 ? `${hours}h ${mins}m` : `${hours}h`;
    }
    return `${minutes}m`;
};

// 解析时间字符串
const parseTime = (timeString) => {
    try {
        timeString = timeString.toLowerCase().trim();
        if (timeString.endsWith('m')) {
            return parseInt(timeString.slice(0, -1)) || 0;
        } else if (timeString.endsWith('h')) {
            return Math.round(parseFloat(timeString.slice(0, -1)) * 60) || 0;
        }
        return 0;
    } catch (e) {
        console.error(`Error parsing time: ${timeString}`);
        return 0;
    }
};

// 遍历任务，提取时间和标签
if (tasks && tasks.length > 0) {
    tasks.forEach(task => {
        // 调试信息
        console.log("Processing task:", task);
        console.log("Task fields:", task.fields);
        console.log("Task tags:", task.tags);
        
        if (task.text.includes('tt::')) {
            const timeMatch = task.text.match(/\(tt::\s*(\d+(\.\d+)?[mh])\)/);
            if (timeMatch && task.tags) {
                const timeSpent = parseTime(timeMatch[1]);
                if (timeSpent > 0) {
                    totalTime += timeSpent;
                    task.tags.forEach(tag => {
                        tagTimeMap[tag] = (tagTimeMap[tag] || 0) + timeSpent;
                    });
                }
            }
        }
    });
}

console.log("tagTimeMap", tagTimeMap);

// 构建表格内容
const result = Object.entries(tagTimeMap)
    .sort((a, b) => b[1] - a[1]) // 按时间降序排列
    .map(([tag, time]) => [
        `${tag}`,
        formatTime(time)
    ]);

// 添加总计行
if (result.length > 0) {
    result.push(['**Total**', `**${formatTime(totalTime)}**`]);
}

// 调试信息
console.log("Result:", result);

// 如果没有结果，显示空表格而不是 "No results" 消息
dv.table(["Tag", "Time"], result.length > 0 ? result : [["No tasks found", ""]]);
```

---

## 📖 Today's Notes

### 🆕 Notes Created Today
```dataviewjs
// 获取当前笔记的创建日期
let currentFile = dv.current().file;
let cday = currentFile.cday.toFormat("yyyy-MM-dd");

// 获取所有笔记并筛选出与当前笔记同一天创建的笔记
let sameDayNotes = dv.pages()
		.where(p => p.file.cday.toFormat("yyyy-MM-dd") == cday)
		.where(p => p.Type != "Meeting");

// 按创建时间排序
sameDayNotes = sameDayNotes.sort(p => p.file.ctime, 'asc');

// 显示结果
dv.list(sameDayNotes.map(p => p.file.link));
```
### ✏️ Notes Modified Today
```dataviewjs
// 获取当前笔记的创建日期
let currentFile = dv.current().file;
let cday = currentFile.cday.toFormat("yyyy-MM-dd");

// 获取所有笔记并筛选出与当前笔记同一天创建的笔记
let sameDayNotes = dv.pages()
		.where(p => p.file.mtime.toFormat("yyyy-MM-dd") == cday)
		.where(p => p.file.cday.toFormat("yyyy-MM-dd") != cday)
		.where(p => p.Type != "Meeting");

// 按创建时间排序
sameDayNotes = sameDayNotes.sort(p => p.file.mtime, 'asc');

// 显示结果
dv.list(sameDayNotes.map(p => p.file.link));
```

---

## 📊 HABIT TRACKER

```dataviewjs
dv.span("**Learning English**")
const calendarData = {
  year: 2024, // (可选) 默认为当前年份
  colors: {
    blue: ["#8cb9ff", "#69a3ff", "#428bff", "#1872ff", "#0058e2"], // (可选) 默认为绿色
    green: ["#c6e48b", "#7bc96f", "#49af5d", "#2e8840", "#196127"],
    red: ["#ff9e82", "#ff7b55", "#ff4dla", "#e73400", "#bd2a00"],
    orange: ["#ffa244", "#fd7f00", "#dd6f00", "#bf6000", "#9b4e00"],
    pink: ["#ff96cb", "#ff70b8", "#ff3a9d", "ee0077", "#c30062"],
    orangeToRed: ["#ffdf04", "#ffbe04", "#ff9a03", "#ff6d02", "#ff2c01"]
  },
  showCurrentDayBorder: true, // (可选) 默认为 true
  defaultEntryIntensity: 4, // (可选) 默认为 4
  intensityScaleStart: 10, // (可选) 默认为最低值
  intensityScaleEnd: 100, // (可选) 默认为最高值
  entries: [] // (必填) 在 DataviewJS 循环中填充
};

// DataviewJS 循环示例
for (let page of dv.pages('"日记"').where(p => p.LearningEnglish)) {
  calendarData.entries.push({
    date: page.file.name, // (必填) 格式为 YYYY-MM-DD
    intensity: page.LearningEnglish, // (必填) 要跟踪的数据，会自动映射颜色强度
    content: "", // (可选) 添加到日期单元格的文本
    color: "orange" // (可选) 参考 calendarData.colors，如果未提供颜色，则使用 colors[0]
  });
}

// 渲染热力图日历
renderHeatmapCalendar(this.container, calendarData);
```

```dataviewjs
dv.span("**Reading**")
const calendarData = {
  year: 2024, // (可选) 默认为当前年份
  colors: {
    blue: ["#8cb9ff", "#69a3ff", "#428bff", "#1872ff", "#0058e2"], // (可选) 默认为绿色
    green: ["#c6e48b", "#7bc96f", "#49af5d", "#2e8840", "#196127"],
    red: ["#ff9e82", "#ff7b55", "#ff4dla", "#e73400", "#bd2a00"],
    orange: ["#ffa244", "#fd7f00", "#dd6f00", "#bf6000", "#9b4e00"],
    pink: ["#ff96cb", "#ff70b8", "#ff3a9d", "ee0077", "#c30062"],
    orangeToRed: ["#ffdf04", "#ffbe04", "#ff9a03", "#ff6d02", "#ff2c01"]
  },
  showCurrentDayBorder: true, // (可选) 默认为 true
  defaultEntryIntensity: 4, // (可选) 默认为 4
  intensityScaleStart: 10, // (可选) 默认为最低值
  intensityScaleEnd: 100, // (可选) 默认为最高值
  entries: [] // (必填) 在 DataviewJS 循环中填充
};

// DataviewJS 循环示例
for (let page of dv.pages('"日记"').where(p => p.Reading)) {
  calendarData.entries.push({
    date: page.file.name, // (必填) 格式为 YYYY-MM-DD
    intensity: page.Reading, // (必填) 要跟踪的数据，会自动映射颜色强度
    content: "", // (可选) 添加到日期单元格的文本
    color: "green" // (可选) 参考 calendarData.colors，如果未提供颜色，则使用 colors[0]
  });
}

// 渲染热力图日历
renderHeatmapCalendar(this.container, calendarData);
```
