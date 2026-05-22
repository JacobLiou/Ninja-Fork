# Ninja 项目 5W2H 概要文档

## 1. 项目概览

Ninja 是一个基于 .NET 8 + WPF 的 Windows 桌面网络运维与故障排查工具集合，支持 IP 扫描、端口扫描、Ping、Traceroute、DNS、Whois、IP Geolocation、远程连接等能力。

参考：
- [README.md](README.md)
- [Ninja/App.xaml.cs](Ninja/App.xaml.cs)
- [Ninja/MainWindow.xaml.cs](Ninja/MainWindow.xaml.cs)

---

## 2. 5W2H 是什么

5W2H 是一种结构化分析方法，核心问题为：
- What：做什么
- Why：为什么做
- Who：谁来做 / 谁使用
- Where：在哪里发生
- When：什么时候发生
- How：如何实现
- How much：成本/规模/约束是多少

可公开检索参考：
- https://en.wikipedia.org/wiki/Five_Ws

在软件架构分析中，5W2H 常用于把“业务目标”和“技术实现”映射到同一张分析框架中。

---

## 3. Ninja 的 5W2H 映射

### 3.1 What（做什么）

提供统一入口的网络管理与问题诊断工具集：
- 应用列表由 ApplicationName 枚举驱动，并在 UI 中按模块切换。
- 快速命令（Run Command）可直接跳转到目标工具模块。

关键代码：
- [Ninja.Models/ApplicationManager.cs](Ninja.Models/ApplicationManager.cs)
- [Ninja/RunCommandManager.cs](Ninja/RunCommandManager.cs)
- [Ninja/MainWindow.xaml.cs#L1012](Ninja/MainWindow.xaml.cs#L1012)
- [Ninja/MainWindow.xaml.cs#L1206](Ninja/MainWindow.xaml.cs#L1206)

### 3.2 Why（为什么做）

解决“网络排障工具分散、切换成本高、上下文难复用”的问题：
- 多协议/多工具统一在一个桌面应用中。
- 支持 Profile（主机/分组）与历史记录复用。
- 支持启动参数、托盘、自启动、更新检查等运维场景。

关键代码：
- [README.md](README.md)
- [Ninja.Settings/CommandLineManager.cs](Ninja.Settings/CommandLineManager.cs)
- [Ninja.Profiles/ProfileManager.cs](Ninja.Profiles/ProfileManager.cs)

### 3.3 Who（谁）

- 主要用户：网络工程师、系统运维人员、IT 支持。
- 系统内部角色：
1. App/MainWindow（应用生命周期与编排）
2. HostViewModel（模块容器）
3. 具体功能 ViewModel（业务执行）
4. Service（外部 API / 网络调用）
5. Settings/Profile Manager（持久化）

关键代码：
- [Ninja/App.xaml.cs](Ninja/App.xaml.cs)
- [Ninja/MainWindow.xaml.cs](Ninja/MainWindow.xaml.cs)
- [Ninja/ViewModels/IPGeolocationHostViewModel.cs](Ninja/ViewModels/IPGeolocationHostViewModel.cs)
- [Ninja/ViewModels/IPGeolocationViewModel.cs](Ninja/ViewModels/IPGeolocationViewModel.cs)

### 3.4 Where（在哪里）

- 运行环境：Windows 本地桌面（WPF）。
- 数据落地位置：
1. 非便携模式：用户文档目录
2. 便携模式：程序目录
- 关键外部接口：ip-api.com（IP 地理与 DNS 解析）

关键代码：
- [Ninja.Settings/SettingsManager.cs#L52](Ninja.Settings/SettingsManager.cs#L52)
- [Ninja.Settings/ConfigurationManager.cs](Ninja.Settings/ConfigurationManager.cs)
- [Ninja.Models/IPApi/IPGeolocationService.cs](Ninja.Models/IPApi/IPGeolocationService.cs)
- [Ninja.Models/IPApi/DNSResolverService.cs](Ninja.Models/IPApi/DNSResolverService.cs)

### 3.5 When（什么时候）

- 启动时：解析命令行、加载/升级设置、初始化本地化、单实例校验、打开主窗体。
- 主窗体渲染后：欢迎向导、加载应用列表、加载 Profiles、检查更新。
- 运行时：用户通过应用列表或 Run Command 切换模块并触发工具执行。

关键代码：
- [Ninja/App.xaml.cs#L98](Ninja/App.xaml.cs#L98)
- [Ninja/MainWindow.xaml.cs#L825](Ninja/MainWindow.xaml.cs#L825)
- [Ninja/MainWindow.xaml.cs#L903](Ninja/MainWindow.xaml.cs#L903)
- [Ninja/MainWindow.xaml.cs#L1924](Ninja/MainWindow.xaml.cs#L1924)

### 3.6 How（如何实现）

整体实现风格：WPF + MVVM + 事件重定向 + 模块化 Host 视图。

实现要点：
1. 应用启动编排
- App 负责初始化和全局策略（设置、线程池、单实例、启动页）。
2. 模块分发
- MainWindow 维护 ApplicationName 到具体 View/HostView 的映射，并进行延迟创建。
3. 快速命令入口
- RunCommandManager 维护命令清单，MainWindow 执行后通过 EventSystem 重定向。
4. 模块内执行
- HostViewModel 负责 Tab 与 Profile；具体 ViewModel 调 Service 执行业务查询。
5. 持久化
- SettingsManager（XML）和 ProfileManager（Profile 文件）负责长期数据。

关键代码：
- [Ninja/App.xaml.cs](Ninja/App.xaml.cs)
- [Ninja/MainWindow.xaml.cs#L1206](Ninja/MainWindow.xaml.cs#L1206)
- [Ninja/RunCommandManager.cs](Ninja/RunCommandManager.cs)
- [Ninja.Models/EventSystem/EventSystem.cs](Ninja.Models/EventSystem/EventSystem.cs)
- [Ninja/ViewModels/IPGeolocationViewModel.cs#L184](Ninja/ViewModels/IPGeolocationViewModel.cs#L184)

### 3.7 How much（成本/规模/约束）

代码中未体现“货币成本”计费模型，但存在可量化约束：
1. 外部 API 速率限制：ip-api.com 使用 X-Rl/X-Ttl 控制窗口，服务端本地有降级处理。
2. 并发资源约束：启动阶段按配置调整 ThreadPool 最小线程数。
3. 状态存储规模：历史记录和配置以本地 XML/Profile 文件保存，规模受配置项和用户数据量影响。

关键代码：
- [Ninja.Models/IPApi/IPGeolocationService.cs#L24](Ninja.Models/IPApi/IPGeolocationService.cs#L24)
- [Ninja/App.xaml.cs#L245](Ninja/App.xaml.cs#L245)
- [Ninja.Settings/SettingsInfo.cs#L3807](Ninja.Settings/SettingsInfo.cs#L3807)
- [Ninja.Settings/SettingsInfo.cs#L3886](Ninja.Settings/SettingsInfo.cs#L3886)

---

## 4. 架构设计图

~~~mermaid
flowchart LR
    U[User] --> MW[MainWindow]

    subgraph UI[UI Layer]
      MW --> AV[Application Views / Host Views]
      AV --> VM[Feature ViewModels]
    end

    subgraph APP[Application Core]
      APPENTRY[App Startup]
      CMD[RunCommandManager]
      EVT[EventSystem]
      AM[ApplicationManager]
    end

    subgraph DOMAIN[Domain & Data]
      PM[ProfileManager]
      SM[SettingsManager]
      CFG[ConfigurationManager]
    end

    subgraph INTEGRATION[Integration]
      SVC[IPGeolocationService / DNSResolverService]
      EXT[ip-api.com / edns.ip-api.com]
    end

    APPENTRY --> MW
    CMD --> MW
    MW --> EVT
    EVT --> MW
    MW --> AM
    VM --> SVC
    SVC --> EXT
    MW --> PM
    MW --> SM
    VM --> SM
    PM --> SM
    SM --> CFG
~~~

---

## 5. 核心业务流

### 5.1 主业务流（应用启动到功能可用）

~~~mermaid
sequenceDiagram
    participant A as App
    participant CL as CommandLineManager
    participant SM as SettingsManager
    participant MW as MainWindow
    participant PM as ProfileManager

    A->>CL: 解析启动参数
    A->>SM: Load/Initialize/Upgrade Settings
    A->>A: 本地化 + 单实例校验 + 线程池配置
    A->>MW: 启动 MainWindow
    MW->>MW: WelcomeThenLoadAsync
    MW->>MW: LoadApplicationList + SetRunCommandsView
    MW->>PM: 加载并刷新 Profiles
    MW->>MW: 系统进入可交互状态
~~~

对应代码：
- [Ninja/App.xaml.cs#L98](Ninja/App.xaml.cs#L98)
- [Ninja/MainWindow.xaml.cs#L825](Ninja/MainWindow.xaml.cs#L825)
- [Ninja/MainWindow.xaml.cs#L903](Ninja/MainWindow.xaml.cs#L903)

### 5.2 工具执行流（以 IPGeolocation 为例）

~~~mermaid
sequenceDiagram
    participant User
    participant MW as MainWindow
    participant RC as RunCommandManager
    participant EVT as EventSystem
    participant HVM as IPGeolocationHostViewModel
    participant VM as IPGeolocationViewModel
    participant SVC as IPGeolocationService
    participant API as ip-api.com

    User->>MW: 打开 Run Command 并选择 IPGeolocation
    MW->>RC: 获取命令列表/命令元数据
    MW->>EVT: RedirectToApplication(IPGeolocation, arg)
    EVT->>MW: OnRedirectDataToApplicationEvent
    MW->>HVM: 显示模块并 AddTab(arg)
    HVM->>VM: 创建新 Tab 对应 ViewModel
    User->>VM: Query
    VM->>SVC: GetIPGeolocationAsync(host)
    SVC->>API: HTTP 请求
    API-->>SVC: JSON + 限流头
    SVC-->>VM: IPGeolocationResult
    VM-->>User: 展示结果 / 状态信息
    VM->>MW: 更新历史与导出状态
~~~

对应代码：
- [Ninja/MainWindow.xaml.cs#L1924](Ninja/MainWindow.xaml.cs#L1924)
- [Ninja/MainWindow.xaml.cs#L1969](Ninja/MainWindow.xaml.cs#L1969)
- [Ninja/MainWindow.xaml.cs#L1637](Ninja/MainWindow.xaml.cs#L1637)
- [Ninja/ViewModels/IPGeolocationHostViewModel.cs#L344](Ninja/ViewModels/IPGeolocationHostViewModel.cs#L344)
- [Ninja/ViewModels/IPGeolocationViewModel.cs#L184](Ninja/ViewModels/IPGeolocationViewModel.cs#L184)
- [Ninja.Models/IPApi/IPGeolocationService.cs#L53](Ninja.Models/IPApi/IPGeolocationService.cs#L53)

---

## 6. 核心数据流

~~~mermaid
flowchart TD
    IN1[用户输入 Host/IP] --> VM[IPGeolocationViewModel]
    VM -->|Query| SVC[IPGeolocationService]
    SVC -->|HTTP JSON| EXT[ip-api.com]
    EXT --> SVC
    SVC -->|IPGeolocationResult| VM
    VM --> UI[结果展示 Result/Status]

    VM -->|更新历史| SET[SettingsManager.Current]
    SET --> XML[Settings.xml]

    PM[ProfileManager] --> PF[Profile files xml/encrypted]
    MW[MainWindow] --> PM
    MW --> SET
~~~

说明：
- 输入数据：主机名/IP、命令参数、Profile 选择。
- 中间数据：Result DTO（含成功、错误、限流状态）。
- 输出数据：UI 展示、历史记录、导出文件（CSV/XML/JSON）。

---

## 7. 重要类图

~~~mermaid
classDiagram
    class App {
      +Application_Startup()
      +Application_Exit()
    }

    class MainWindow {
      -LoadApplicationList()
      -OnApplicationViewVisible(name)
      -RunCommandDo()
      -EventSystem_RedirectDataToApplicationEvent()
    }

    class RunCommandManager {
      +GetList()
      -GetApplicationList()
      -GetSettingsList()
    }

    class EventSystem {
      +OnRedirectDataToApplicationEvent
      +RedirectToApplication(application, data)
      +OnRedirectToSettingsEvent
      +RedirectToSettings()
    }

    class IPGeolocationHostViewModel {
      +AddTabCommand
      -AddTab(domain)
      -SetProfilesView()
    }

    class IPGeolocationViewModel {
      +QueryCommand
      +ExportCommand
      -Query()
      -AddHostToHistory(host)
    }

    class IPGeolocationService {
      +GetIPGeolocationAsync(host)
      -IsInRateLimit()
      -UpdateRateLimit(headers)
    }

    class SettingsManager {
      +Current
      +Load()
      +Save()
      +Upgrade(from,to)
    }

    class ProfileManager {
      +Groups
      +Switch(...)
      +Save()
    }

    App --> MainWindow
    MainWindow --> RunCommandManager
    MainWindow --> EventSystem
    MainWindow --> IPGeolocationHostViewModel
    IPGeolocationHostViewModel --> IPGeolocationViewModel
    IPGeolocationViewModel --> IPGeolocationService
    MainWindow --> SettingsManager
    MainWindow --> ProfileManager
~~~

---

## 8. 实现与质量观察

### 8.1 架构优点

1. 模块化清晰：ApplicationName + HostViewModel 模式使功能扩展路径稳定。
2. 解耦跳转：EventSystem 将命令入口与 UI 切换逻辑解耦。
3. 持久化可控：Settings 与 Profile 分层，便于迁移和升级。

### 8.2 风险点（建议优先检查）

IPGeolocation 历史记录写入目标疑似错误：
- 读取源是 IPGeolocation_HostHistory
- 清空目标也是 IPGeolocation_HostHistory
- 但回写使用了 Whois_DomainHistory

定位：
- [Ninja/ViewModels/IPGeolocationViewModel.cs#L300](Ninja/ViewModels/IPGeolocationViewModel.cs#L300)
- [Ninja/ViewModels/IPGeolocationViewModel.cs#L304](Ninja/ViewModels/IPGeolocationViewModel.cs#L304)
- [Ninja/ViewModels/IPGeolocationViewModel.cs#L308](Ninja/ViewModels/IPGeolocationViewModel.cs#L308)
- [Ninja.Settings/SettingsInfo.cs#L3886](Ninja.Settings/SettingsInfo.cs#L3886)
- [Ninja.Settings/SettingsInfo.cs#L3807](Ninja.Settings/SettingsInfo.cs#L3807)

该问题会导致 IPGeolocation 历史可能无法正确回填，且污染 Whois 历史数据。

---

## 9. 结论

从 5W2H 视角看，Ninja 的实现是典型的“桌面聚合式网络工具平台”：
- 目标明确（What/Why）
- 角色分层清晰（Who）
- 运行与存储边界明确（Where/When）
- 技术实现可维护（How）
- 资源与约束主要体现在 API 限流、线程与本地数据规模（How much）

因此，该项目适合持续按“新增 ApplicationName + HostViewModel + FeatureViewModel + Service”模板演进。