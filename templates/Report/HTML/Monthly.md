<%*
// 设置文件名和路径
const start = moment().subtract(1, 'month').startOf('month');
const end = moment(start).endOf('month');
const monthNumber = start.format('MM');
const fileName = `${start.format('YYYY.MM')} (第 ${monthNumber} 月)`;
const year = start.format('YYYY');
const yearFolderPath = `工作报告/月报/${year}`;
const targetFilePath = `${yearFolderPath}/${fileName}.md`;

// 检查目标路径是否存在文件
const currentFilePath = tp.file.path(true);
const targetFile = app.vault.getAbstractFileByPath(targetFilePath);

if (targetFile) {
    // 如果目标文件已存在，且不是当前文件
    if (currentFilePath !== targetFile.path) {
        const currentFile = app.vault.getAbstractFileByPath(currentFilePath);
        if (currentFile) {
            await app.vault.delete(currentFile);
            await app.workspace.openLinkText(fileName, targetFile.path);
        }
        return;
    }
} else {
    // 确保年份文件夹存在
    const yearFolder = app.vault.getAbstractFileByPath(yearFolderPath);
    if (!yearFolder) {
        await app.vault.createFolder(yearFolderPath);
    }

    // 重命名并移动文件
    const currentFile = app.vault.getAbstractFileByPath(currentFilePath);
    if (currentFile) {
        try {
            await app.fileManager.renameFile(currentFile, targetFilePath);
        } catch (error) {
            console.error('移动文件失败:', error);
            // 如果移动失败，至少确保文件名正确
            await tp.file.rename(fileName);
        }
    }
}
_%>
---
created: <% tp.file.creation_date() %>
month: <% moment().subtract(1, 'month').format("YYYY-MM") %>
cssclass: monthly-report
---

<%*
// 获取上月的开始和结束日期
const monthStart = moment().subtract(1, 'month').startOf('month');
const monthEnd = moment(monthStart).endOf('month');

// 日期提取函数
function extractDate(filename) {
    // 支持多种日期格式
    const patterns = [
        /^(\d{4}-\d{2}-\d{2})/,  // 2024-03-25
        /^(\d{4}\.\d{2}\.\d{2})/, // 2024.03.25
        /(\d{4})\s*[年\.]\s*(\d{1,2})\s*[月\.]\s*(\d{1,2})/ // 2024年03月25日 或 2024.11.25
    ];
    
    for (const pattern of patterns) {
        const match = filename.match(pattern);
        if (match) {
            if (match.length === 4) {
                // 处理年月日格式
                const [_, year, month, day] = match;
                return `${year}-${month.padStart(2, '0')}-${day.padStart(2, '0')}`;
            } else if (match[1]) {
                // 处理已经是标准格式的日期
                return match[1].replace(/\./g, '-');
            }
        }
    }
    return null;
}

// 搜索会议记录
const meetingPath = '工作笔记/工作记录/会议记录';
const meetingFiles = app.vault.getFiles()
    .filter(f => f.path.startsWith(meetingPath))
    .map(async f => {
        const content = await app.vault.read(f);
        const date = extractDate(f.basename);
        
        // 提取会议息
        const summaryMatch = content.match(/\(summary::\s*([^)]+)\)/);
        
        // 从 frontmatter 中提取会议类和参会人
        const frontmatterMatch = content.match(/^---\n([\s\S]*?)\n---/);
        let meetingType = '未分类';
        let attendees = '未记录';
        
        if (frontmatterMatch) {
            const frontmatter = frontmatterMatch[1];
            // 提取会议类型
            const typeMatch = frontmatter.match(/MeetingType:\s*(.+)$/m);
            if (typeMatch) {
                meetingType = typeMatch[1].trim();
            }
            // 提取参会人 - 处理YAML数组格式
            const attendeesMatch = frontmatter.match(/Attendees:\s*\n((?:\s*-\s*.+\n?)*)/m);
            if (attendeesMatch) {
                // 处理YAML数组格式的参会人列表
                attendees = attendeesMatch[1]
                    .split('\n')
                    .map(line => line.replace(/^\s*-\s*/, '').trim()) // 移除 YAML 列表符号
                    .filter(name => name) // 过滤空值
                    .join('、');
            }
        }
        
        return {
            file: f,
            date,
            summary: summaryMatch ? summaryMatch[1].trim() : '无摘要',
            meetingType,
            participants: attendees
        };
    });

const meetings = await Promise.all(meetingFiles);
const monthMeetings = meetings.filter(m => 
    m.date && moment(m.date).isBetween(monthStart, monthEnd, 'day', '[]')
);

// 获取任务
const taskFiles = app.vault.getFiles()
    .filter(f => {
        const isDaily = f.path.includes('日记');
        const isMeeting = f.path.includes('会议记录');
        if (!isDaily && !isMeeting) return false;
        
        const date = extractDate(f.basename);
        if (date) {
            return moment(date).isBetween(monthStart, monthEnd, 'day', '[]');
        }
        return false;
    });

// 获取任务列表并分类
const completedTasks = [];
const pendingTasks = [];
for (const file of taskFiles) {
    const content = await app.vault.read(file);
    // 匹配未完成的任务
    const pendingMatches = content.match(/- \[ \].+/g);
    if (pendingMatches) {
        pendingTasks.push(...pendingMatches.map(task => ({
            text: task.replace('- [ ]', '').trim(),
            source: file.basename,
            date: extractDate(file.basename),
            link: file.path
        })));
    }
    // 匹配已完成的任务
    const completedMatches = content.match(/- \[x\].+/g);
    if (completedMatches) {
        completedTasks.push(...completedMatches.map(task => ({
            text: task.replace('- [x]', '').trim(),
            source: file.basename,
            date: extractDate(file.basename),
            link: file.path
        })));
    }
}

// 获取本月文档
const docs = app.vault.getFiles()
    .filter(f => {
        const fileDate = moment(f.stat.ctime);
        return fileDate.isBetween(monthStart, monthEnd, 'day', '[]');
    })
    .filter(f => !f.path.includes('会议记录') && !f.path.includes('日记') && !f.path.includes('attachments'))
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
tR += `# ${monthStart.format("YYYY年MM月")}月报
<div class="date-range">${monthStart.format('YYYY/MM/DD')} - ${monthEnd.format('YYYY/MM/DD')}</div>

## 📅 本月会议 <span class="count-badge">${monthMeetings.length}</span>
<div class="meetings-grid">
${monthMeetings.length > 0 
    ? monthMeetings.map(meeting => `
<div class="meeting-card">
    <div class="meeting-header">
        <div class="meeting-date">${meeting.date}</div>
        <div class="meeting-type">${meeting.meetingType}</div>
    </div>
    <div class="meeting-title">${meeting.file.basename}</div>
    <div class="meeting-summary">${meeting.summary}</div>
    <div class="meeting-participants">
        <span class="participants-label">参会人：</span>${meeting.participants}
    </div>
</div>`).join('\n')
    : '> [!note] 本月无会议记录'}
</div>

## ✅ 本月任务 <span class="count-badge">${pendingTasks.length + completedTasks.length}</span>
<div class="tasks-container">
    <div class="tasks-section pending">
        <h3>📋 待处理任务 <span class="count-badge amber">${pendingTasks.length}</span></h3>
        <div class="tasks-grid">
        ${pendingTasks.length > 0
            ? pendingTasks.map(task => `
            <div class="task-card pending">
                <div class="task-date">${task.date}</div>
                <div class="task-text">${task.text}</div>
                <div class="task-source">来源: [[${task.source}]]</div>
            </div>`).join('\n')
            : '> [!note] 无待处理任务'}
        </div>
    </div>
    
    <div class="tasks-section completed">
        <h3>✨ 已完成任务 <span class="count-badge green">${completedTasks.length}</span></h3>
        <div class="tasks-grid">
        ${completedTasks.length > 0
            ? completedTasks.map(task => `
            <div class="task-card completed">
                <div class="task-date">${task.date}</div>
                <div class="task-text">${task.text}</div>
                <div class="task-source">来源: [[${task.source}]]</div>
            </div>`).join('\n')
            : '> [!note] 无已完成任务'}
        </div>
    </div>
</div>

## 📝 本月文档 <span class="count-badge">${docs.length}</span>
<div class="docs-grid">
${docs.length > 0
    ? docs.map(doc => `
    <div class="doc-card">
        <div class="doc-date">${doc.date}</div>
        <div class="doc-title">${doc.title}</div>
        ${doc.summary ? `<div class="doc-summary">${doc.summary}</div>` : ''}
        <div class="doc-path">${doc.path}</div>
    </div>`).join('\n')
    : '> [!note] 本月无新文档'}
</div>

## 📊 月度总结
<div class="report-summary">
    <div class="summary-overview">
        <div class="stat-card meetings">
            <div class="stat-icon">📅</div>
            <div class="stat-number">${monthMeetings.length}</div>
            <div class="stat-label">会议数</div>
        </div>
        <div class="stat-card tasks-completed">
            <div class="stat-icon">✅</div>
            <div class="stat-number">${completedTasks.length}</div>
            <div class="stat-label">已完成任务</div>
        </div>
        <div class="stat-card tasks-pending">
            <div class="stat-icon">⏳</div>
            <div class="stat-number">${pendingTasks.length}</div>
            <div class="stat-label">待处理任务</div>
        </div>
        <div class="stat-card docs">
            <div class="stat-icon">📝</div>
            <div class="stat-number">${docs.length}</div>
            <div class="stat-label">文档数</div>
        </div>
    </div>

    <div class="summary-section key-works">
        <h3>💫 重点工作</h3>
        ${completedTasks.length > 0 
            ? `<div class="key-works-list">` + 
              completedTasks.slice(0, 5).map(task => 
                `<div class="key-work-item">
                    <span class="work-date">${task.date}</span>
                    <span class="work-text">${task.text}</span>
                </div>`
              ).join('') + 
              `</div>`
            : '<div class="empty-notice">本月无重点工作完成</div>'}
    </div>

    <div class="summary-section progress">
        <h3>📈 工作进展</h3>
        <div class="progress-stats">
            <div class="progress-item">
                <div class="progress-label">任务完成率</div>
                <div class="progress-bar">
                    <div class="progress-fill" style="width: ${Math.round(completedTasks.length / (completedTasks.length + pendingTasks.length) * 100 || 0)}%">
                        ${Math.round(completedTasks.length / (completedTasks.length + pendingTasks.length) * 100 || 0)}%
                    </div>
                </div>
            </div>
        </div>
    </div>

    <div class="summary-section issues">
        <h3>⚠️ 问题与建议</h3>
        <div class="issues-grid">
            <div class="issue-card ${pendingTasks.length > 5 ? 'warning' : 'good'}">
                <div class="issue-icon">${pendingTasks.length > 5 ? '⚠️' : '✅'}</div>
                <div class="issue-title">任务积压</div>
                <div class="issue-status">
                    ${pendingTasks.length > 5 ? '任务积压较多' : '任务量正常'}
                </div>
            </div>
            <div class="issue-card ${monthMeetings.length < 4 ? 'warning' : 'good'}">
                <div class="issue-icon">${monthMeetings.length < 4 ? '⚠️' : '✅'}</div>
                <div class="issue-title">会议参与</div>
                <div class="issue-status">
                    ${monthMeetings.length < 4 ? '会议参与较少' : '会议参与正常'}
                </div>
            </div>
        </div>
        <div class="suggestions">
            <h4>📝 改进建议</h4>
            <ul class="suggestion-list">
                ${pendingTasks.length > 5 
                    ? `<li>建议合理规划任务优先级，加快任务处理进度</li>
                       <li>可考虑任务授权或寻求团队协助</li>`
                    : `<li>继续保持当前的工作节奏</li>
                       <li>建议做好任务提前规划，避免临期压力</li>`}
            </ul>
        </div>
    </div>

    <div class="summary-section next-focus">
        <h3>🎯 下月工作重点</h3>
        ${pendingTasks.length > 0 
            ? `<div class="next-tasks">` +
              pendingTasks.slice(0, 3).map(task => 
                `<div class="next-task-item">
                    <div class="task-priority">P${pendingTasks.indexOf(task) + 1}</div>
                    <div class="task-content">${task.text}</div>
                    <div class="task-meta">
                        <span class="task-date">${task.date}</span>
                        <span class="task-source">来源: [[${task.source}]]</span>
                    </div>
                </div>`
              ).join('') +
              `</div>`
            : '<div class="empty-notice">暂无待处理的重点任务</div>'}
    </div>
</div>`;
_%>