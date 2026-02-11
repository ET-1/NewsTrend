# NewsTrend 部署指南

## 🎯 工作原理

### 两层调度机制

```
┌─────────────────────────────────────────────────────────┐
│  第一层：GitHub Actions Cron（程序唤醒）                  │
│  ├─ 定义在：.github/workflows/crawler.yml               │
│  ├─ 作用：定期唤醒程序运行                               │
│  └─ 示例：每小时第33分钟运行一次                         │
└─────────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────────┐
│  第二层：Timeline 调度器（程序内部逻辑）                  │
│  ├─ 定义在：config/timeline.yaml                        │
│  ├─ 作用：根据当前时间段决定做什么                       │
│  └─ 控制：推送/AI分析/报告模式                           │
└─────────────────────────────────────────────────────────┘
```

## ⚙️ 调度配置

### 1. GitHub Actions 触发频率

**文件位置**：`.github/workflows/crawler.yml`

```yaml
on:
  schedule:
    - cron: "33 * * * *"  # 每小时第33分钟
```

**常用配置**：
```yaml
# 每小时运行
- cron: "33 * * * *"

# 每2小时运行
- cron: "0 */2 * * *"

# 每天运行一次（早上9点 北京时间）
- cron: "0 1 * * *"

# 工作时间每小时运行（北京时间 9:00-18:00）
- cron: "0 1-10 * * *"
```

### 2. Timeline 调度策略

**文件位置**：`config/config.yaml`

```yaml
schedule:
  enabled: true
  preset: "office_hours"  # 选择预设模板
```

**可用预设**：
- `always_on` - 全天候，有新增立即推送
- `morning_evening` - 全天推送 + 晚间汇总（推荐）
- `office_hours` - 工作日三段式（当前配置）
- `night_owl` - 午后速览 + 深夜汇总
- `custom` - 完全自定义

## 📋 当前配置（office_hours）

### 工作日模式
```
09:00-11:00  到岗速览    → current榜单 + AI分析（仅一次）
13:00-15:00  午间热点    → current榜单（仅一次）
17:00-19:00  收工汇总    → daily汇总 + AI分析（仅一次）
其他时间     静默采集    → 只采集数据，不推送
```

### 周末模式
```
10:00-19:00  周末自由    → current榜单推送（不限次数）
其他时间     静默采集    → 只采集数据，不推送
```

## 🚀 快速调整

### 场景1：改为每2小时运行一次

编辑 `.github/workflows/crawler.yml`：
```yaml
- cron: "0 */2 * * *"
```

### 场景2：改为全天候推送

编辑 `config/config.yaml`：
```yaml
schedule:
  preset: "always_on"
```

### 场景3：改为早晚汇总

编辑 `config/config.yaml`：
```yaml
schedule:
  preset: "morning_evening"
```

## 📊 查看当前调度状态

运行命令：
```bash
python -m trendradar --show-schedule
```

输出示例：
```
============================================================
TrendRadar v6.0.0 调度状态
============================================================

⏰ 当前时间: 2026-02-11 09:07:23 (Asia/Shanghai)
📅 当前日期: 2026年02月11日

📋 调度信息:
  日计划: workday
  当前时间段: 到岗速览 (morning_overview)

🔧 行为开关:
  采集数据: ✅ 是
  AI 分析:  ✅ 是
  推送通知: ✅ 是
  报告模式: daily
  AI 模式:  daily

🔁 一次性控制:
  AI 分析:  仅一次 (今日未执行 ✅)
  推送通知: 仅一次 (今日未执行 ✅)
============================================================
```

## 🌐 GitHub Pages 配置

### 设置步骤
1. 进入仓库 Settings → Pages
2. Source 选择：`master` 分支
3. Folder 选择：`/ (root)`
4. 点击 Save

### 访问地址
```
https://ET-1.github.io/NewsTrend/
```

## 📝 提交修改

```bash
cd /home/ivan/code/NewsTrend

# 添加修改
git add .github/workflows/crawler.yml config/

# 提交
git commit -m "配置调度策略"

# 推送
git push origin master
```

## 💡 建议

1. **GitHub Actions 唤醒频率**：建议至少每1-2小时运行一次
2. **Timeline 推送策略**：根据个人作息调整 `config.yaml` 中的 preset
3. **测试**：使用 `workflow_dispatch` 手动触发测试
4. **监控**：在 Actions 页面查看运行日志

---

更多详细配置请参考：
- `config/timeline.yaml` - 完整的时间段配置说明
- `config/config.yaml` - 主配置文件
- 项目 README.md - 完整功能说明
