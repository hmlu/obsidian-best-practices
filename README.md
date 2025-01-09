# **快速开始**

欢迎使用我的 Obsidian 最佳实践项目！通过此项目，你可以快速配置并使用 Obsidian，从而显著提升日记记录、知识管理和工作效率。

## **项目亮点**

- **开箱即用的插件配置**：精心挑选并预配置的 Obsidian 插件，简化安装流程，省去繁琐的设置步骤。
- **丰富的模板**：涵盖多种场景的实用模板，包括日记、会议记录、周报、月报、季报，以及英文学习笔记等。
- **强大的功能支持**：内置会议记录、任务管理和时间管理功能，帮助你高效规划和跟踪工作进度。
- **时间管理统计**：日记模板中集成了时间管理统计功能，让你轻松分析和优化每日安排。

通过我的 Obsidian 最佳实践项目，你无需从零开始配置，只需下载并简单设置，即可快速上手，享受高效便捷的 Obsidian 使用体验！

---

## **1. 安装 Obsidian**

1. **下载 Obsidian 客户端**  
    请访问 Obsidian 官方下载页面，根据你的设备选择适合的版本进行下载：  
    [Obsidian 下载地址](https://obsidian.md/download)
    
2. **安装 Obsidian 客户端**  
    下载完成后，根据系统提示完成安装。Obsidian 支持 Windows、macOS、Linux 和移动端（iOS/Android）。
    
3. **创建或导入工作空间（Vault）**
    
    - 如果你是新用户：点击 **“Create a new vault”**，输入工作空间名称并选择存储路径。
    - 如果你已有现成的模板和文件：点击 **“Open folder as vault”**，选择包含配置和模板的文件夹。

更多详细的安装说明，请参考 Obsidian 官方文档：  [Obsidian 官方文档](https://obsidian.md/)

---

## **2. 下载项目文件**

1. **克隆仓库或下载压缩包**  
    在终端执行以下命令克隆此仓库，或直接下载压缩包解压：
    
	```bash
	git clone https://github.com/hmlu/obsidian-best-practices.git
	```
    
    或访问 [GitHub 仓库](https://github.com/hmlu/obsidian-best-practices) 下载最新版本。
    
2. **复制文件到 Obsidian Vault 文件夹**  
    将下载的 `templates` 和 `.obsidian` 目录复制到你的 Obsidian 仓库根目录中。

---

## **3. 配置插件和模板**

1. **启用社区插件**  
    打开 Obsidian，点击左下角齿轮图标进入 **“Settings”**：
    
    - 前往 **Community Plugins** 页面，启用 **Safe Mode**（安全模式）。
2. **使用插件和模板**
    - 将下载的  `templates` 和 `.obsidian` 目录复制到你的 Obsidian 仓库根目录后，你将立即拥有我开发的所有模板、精心挑选的插件以及完整的插件配置。
3. **Remotely Save 插件说明**
	- **Remotely Save** 是一个强大的 Obsidian 插件，用于将你的仓库文件同步到远程存储服务，避免本地文件丢失、以及多设备同步的功能。在使用此插件之前，你需要提前准备一个 S3 存储桶，与对应有权限读写此存储桶的 AK/SK，支持 AWS S3，也支持国内各大云厂商的对象存储服务。

---

## **4. 开始使用**

1. **创建日记**
    
    - 点击 Obsidian 窗口右侧的 ”日历“ 插件中的日期，可为你选中的日期创建一篇日记，并自动按照 `Daily.md` 模板来创建日记。
2. **探索最佳实践**
    
    - 结合模板和插件，快速记录会议、任务，管理知识和内容。
    - 项目中的模板覆盖常用场景，如日记、会议记录、报告等，帮助你高效管理工作。

---

## **项目目录结构说明**

```plaintext
obsidian-best-practices/
├── templates/                # 模板文件夹
│   ├── Daily.md              # 日常笔记模板
│   ├── English Notes.md      # 英语笔记模板
│   ├── Insert Callouts.md    # 插入提醒块模板
│   ├── Meeting.md            # 会议记录模板
│   └── Report/               # 报告模板
│       ├── HTML/             # HTML 格式的报告模板
│       │   ├── Monthly.md    # 月度报告模板
│       │   ├── Quarterly.md  # 季度报告模板
│       │   └── Weekly.md     # 周报模板
│       └── Markdown/         # Markdown 格式的报告模板
│           ├── Monthly.md    
│           ├── Quarterly.md  
│           └── Weekly.md    
├── .obsidian/                # 推荐插件配置
└── README.md                 # 项目说明文档
```

---

## **问题反馈与贡献**

如果你在使用中遇到问题，或有好的建议，欢迎通过 GitHub 提交 Issue 或 Pull Request！  
[GitHub 仓库](https://github.com/hmlu/obsidian-best-practices)

