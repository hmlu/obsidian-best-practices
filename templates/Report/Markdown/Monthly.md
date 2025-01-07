<%*
// 设置文件名和路径
const start = moment().subtract(1, 'month').startOf('month');
const end = moment(start).endOf('month');
const monthNumber = start.format('MM');
const fileName = `${start.format('YYYY.MM')} (第 ${monthNumber} 月)`;
const year = start.format('YYYY');
const yearFolderPath = `工作报告（Markdown）/月报/${year}`;
const targetFilePath = `${yearFolderPath}/${fileName}.md`;

// 检查目标路径是否存在文件
const currentFilePath = tp.file.path(true);
const targetFile = app.vault.getAbstractFileByPath(targetFilePath);

if (targetFile) {
    if (currentFilePath !== targetFile.path) {
        const currentFile = app.vault.getAbstractFileByPath(currentFilePath);
        if (currentFile) {
            await app.vault.delete(currentFile);
            await app.workspace.openLinkText(fileName, targetFile.path);
        }
        return;
    }
} else {
    const yearFolder = app.vault.getAbstractFileByPath(yearFolderPath);
    if (!yearFolder) {
        await app.vault.createFolder(yearFolderPath);
    }

    const currentFile = app.vault.getAbstractFileByPath(currentFilePath);
    if (currentFile) {
        try {
            await app.fileManager.renameFile(currentFile, targetFilePath);
        } catch (error) {
            console.error('移动文件失败:', error);
            await tp.file.rename(fileName);
        }
    }
}

// 获取本月的会议记录、任务和文档
const meetingPath = '工作笔记/工作记录/会议记录';
const meetings = await Promise.all(
    app.vault.getFiles()
        .filter(f => f.path.startsWith(meetingPath))
        .map(async f => {
            const content = await app.vault.read(f);
            const date = f.basename.match(/^\d{4}[-\.]\d{2}[-\.]\d{2}/)?.[0]?.replace(/\./g, '-');
            
            const summaryMatch = content.match(/\(summary::\s*([^)]+)\)/);
            const frontmatterMatch = content.match(/^---\n([\s\S]*?)\n---/);
            let meetingType = '未分类';
            let attendees = '未记录';
            
            if (frontmatterMatch) {
                const frontmatter = frontmatterMatch[1];
                const typeMatch = frontmatter.match(/MeetingType:\s*(.+)$/m);
                if (typeMatch) meetingType = typeMatch[1].trim();
                
                const attendeesMatch = frontmatter.match(/Attendees:\s*\n((?:\s*-\s*.+\n?)*)/m);
                if (attendeesMatch) {
                    attendees = attendeesMatch[1]
                        .split('\n')
                        .map(line => line.replace(/^\s*-\s*/, '').trim())
                        .filter(Boolean)
                        .join('、');
                }
            }
            
            return {
                title: f.basename,
                date,
                summary: summaryMatch ? summaryMatch[1].trim() : '无摘要',
                meetingType,
                participants: attendees
            };
        })
);

const monthMeetings = meetings.filter(m => 
    m.date && moment(m.date).isBetween(start, end, 'day', '[]')
);

// 获取任务
const taskFiles = app.vault.getFiles()
    .filter(f => {
        const isDaily = f.path.includes('日记');
        const isMeeting = f.path.includes('会议记录');
        if (!isDaily && !isMeeting) return false;
        
        const date = f.basename.match(/^\d{4}[-\.]\d{2}[-\.]\d{2}/)?.[0]?.replace(/\./g, '-');
        return date && moment(date).isBetween(start, end, 'day', '[]');
    });

const completedTasks = [];
const pendingTasks = [];

for (const file of taskFiles) {
    const content = await app.vault.read(file);
    
    const pendingMatches = content.match(/- \[ \].+/g) || [];
    pendingTasks.push(...pendingMatches.map(task => ({
        text: task.replace(/^-\s*\[\s*\]\s*/, '').trim(),
        source: file.basename,
        date: file.basename.match(/^\d{4}[-\.]\d{2}[-\.]\d{2}/)?.[0]?.replace(/\./g, '-')
    })));
    
    const completedMatches = content.match(/- \[x\].+/g) || [];
    completedTasks.push(...completedMatches.map(task => ({
        text: task.replace(/^-\s*\[x\]\s*/, '').trim(),
        source: file.basename,
        date: file.basename.match(/^\d{4}[-\.]\d{2}[-\.]\d{2}/)?.[0]?.replace(/\./g, '-')
    })));
}

// 获取本月文档
const docs = app.vault.getFiles()
    .filter(f => {
        const fileDate = moment(f.stat.ctime);
        return fileDate.isBetween(start, end, 'day', '[]') &&
               !f.path.includes('会议记录') && 
               !f.path.includes('日记') && 
               !f.path.includes('attachments');
    })
    .map(f => ({
        title: f.basename,
        path: f.path,
        date: moment(f.stat.ctime).format('YYYY-MM-DD'),
        summary: (() => {
            try {
                const content = app.vault.read(f);
                const summaryMatch = content.match(/\(summary::\s*([^)]+)\)/);
                return summaryMatch ? summaryMatch[1].trim() : '';
            } catch {
                return '';
            }
        })()
    }));

// 渲染模板
tR += `---
created: ${tp.file.creation_date()}
month: ${start.format("YYYY-MM")}
cssclass: monthly-report
---

# 月报 - ${start.format("YYYY年MM月")}

> [!info] **时间范围**
> ${start.format('YYYY/MM/DD')} - ${end.format('YYYY/MM/DD')}

## 📅 本月会议 (${monthMeetings.length})

${monthMeetings.length > 0 
    ? monthMeetings.map(meeting => `
> [!note]+ **${meeting.date} ${meeting.meetingType}**
> - **主题**：[[${meeting.title}]]
> - **摘要**：${meeting.summary}
> - **参会人**：${meeting.participants}`).join('\n\n')
    : '> [!warning] 本月无会议记录'}

## ✅ 本月任务 (${pendingTasks.length + completedTasks.length})

### 📋 待处理任务 (${pendingTasks.length})
${pendingTasks.length > 0
    ? pendingTasks.map(task => `- ${task.text}\n  📅 ${task.date} | 📎 [[${task.source}]]`).join('\n')
    : '> [!warning] 无待处理任务'}

### ✨ 已完成任务 (${completedTasks.length})
${completedTasks.length > 0
    ? completedTasks.map(task => `- ${task.text}\n  📅 ${task.date} | 📎 [[${task.source}]]`).join('\n')
    : '> [!warning] 无已完成任务'}

## 📝 本月文档 (${docs.length})

${docs.length > 0
    ? docs.map(doc => `### [[${doc.title}]]
- 📅 **创建日期**：${doc.date}
- 📂 **路径**：${doc.path}${doc.summary ? '\n- 📌 **摘要**：' + doc.summary : ''}`).join('\n\n')
    : '> [!warning] 本月无新文档'}

## 📊 月度总结

### 工作概述
> [!summary] 本月工作量统计
> - 参与会议：${monthMeetings.length} 场
> - 完成任务：${completedTasks.length} 项
> - 待处理任务：${pendingTasks.length} 项
> - 文档输出：${docs.length} 份

### 重点工作
${completedTasks.length > 0 
    ? '主要完成了以下工作：\n' + completedTasks.slice(0, 5).map(task => 
        `- ${task.text}`).join('\n')
    : '本月无重点工作完成'}

### 工作进展
- **已完成工作**：完成率 ${Math.round(completedTasks.length / (completedTasks.length + pendingTasks.length) * 100 || 0)}%
- **待处理工作**：剩余 ${pendingTasks.length} 项任务需要跟进

### 问题与建议
- **存在的问题**
  - 待处理任务积压情况：${pendingTasks.length > 5 ? '⚠️ 任务积压较多' : '✅ 任务量正常'}
  - 会议参与情况：${monthMeetings.length < 4 ? '⚠️ 会议参与较少' : '✅ 会议参与正常'}

- **改进建议**
  ${pendingTasks.length > 5 
    ? '- 建议合理规划任务优先级，加快任务处理进度\n  - 可考虑任务授权或寻求团队协助'
    : '- 继续保持当前的工作节奏\n- 建议做好任务提前规划，避免临期压力'}

### 下月工作重点
${pendingTasks.length > 0 
    ? '需要重点关注以下任务：\n' + pendingTasks.slice(0, 3).map(task => 
        `- ${task.text}`).join('\n')
    : '暂无待处理的重点任务'}`;
_%>