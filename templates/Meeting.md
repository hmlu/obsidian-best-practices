---
<%*
let title = tp.file.title;
if (title.startsWith("Untitled")) {
 title = await tp.system.prompt("Enter the title of the meeting");
 if(!title) return;
}
if (title == "") {
title = "Untitled";
} 

let filename = tp.file.creation_date("YYYY.MM.DD") + " " + title;
await tp.file.rename(title);
await tp.file.move("/工作笔记/工作记录/会议记录/" + filename)
-%>
Type: Meeting
Date: <% tp.file.creation_date("YYYY-MM-DD") %>
MeetingType: Internal
Client: 
Attendees: 
tags: []
---
## Summary

(summary:: <% title %>)

## Logs

## Tasks
