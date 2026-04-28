# NexT 文章目录 (TOC) 实施计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**目标：** 在山河博客启用 NexT 内置目录功能，提供侧边栏固定目录（桌面端）和文中折叠目录（移动端）。

**架构：** 通过 NexT 主题配置文件启用内置 TOC，利用主题 CSS 变量定制样式，无需额外插件。

**技术栈：** NexT v8.27.0、Hexo、YAML 配置

---

## 文件变更概览

| 文件 | 操作 |
|------|------|
| `source/_config.next.yml` | 添加 toc 配置段 |
| `source/_data/next.yml` (若存在) | 检查是否有冲突配置 |

---

## 实施任务

### Task 1: 确认 NexT TOC 配置结构

**文件：**
- 查看: `source/_config.next.yml` 当前 `toc` 相关配置
- 参考: NexT v8 官方文档 `toc` 配置项

- [ ] **Step 1: 检查当前配置文件**

读取 `source/_config.next.yml`，确认是否已有 `toc` 配置段。

```bash
grep -n "toc" source/_config.next.yml
```

预期结果：无 `toc` 相关配置（需新增）

---

### Task 2: 添加 TOC 配置

**文件：**
- 修改: `source/_config.next.yml`

- [ ] **Step 1: 添加 toc 配置段**

在 `source/_config.next.yml` 末尾添加：

```yaml
# ---------------------------------------------------------------
# Table of Contents (TOC)
# ---------------------------------------------------------------
toc:
  enable: true
  number: true
  max_depth: 3
  min_depth: 1
  place: right
```

验证：运行 `pnpm hexo clean && pnpm build`，确认无配置错误。

> **注意：** 若 NexT 版本不支持 `place` 参数，目录位置由主题侧边栏配置决定，桌面端自动显示在右侧。
>
> **CSS 样式定制（P1 延期）：** 目录宽度、字体大小等样式定制待核心功能验证后再实施。

---

### Task 3: 验证桌面端显示

**文件：**
- 测试: 启动本地服务器查看效果

- [ ] **Step 1: 启动本地开发服务器**

```bash
cd /Users/nexlume/AI-Workspace/blog
pnpm hexo server
```

- [ ] **Step 2: 验证目录提取和显示**

1. 打开浏览器访问 http://localhost:4000/shanhe/
2. 进入一篇文章（如「Superpowers用户手册」）
3. 确认右侧边栏显示目录列表
4. 确认目录显示 **h2（##）和 h3（###）** 标题
5. 确认目录显示章节编号
6. 确认点击目录项跳转到对应章节
7. **滚动文章**，观察目录高亮是否跟随当前章节

预期结果：
- 侧边栏显示文章各章节（h2、h3 层级）
- 目录项带数字编号 (1, 1.1, 1.1.1 等)
- 滚动时**当前章节高亮**（NexT 内置行为）

---

### Task 4: 验证移动端适配

**文件：**
- 测试: 响应式布局检查

- [ ] **Step 1: 测试移动端目录展开/折叠行为**

1. 打开浏览器开发者工具 (F12)
2. 切换到移动端视图 (iPhone 12 / 375px 宽度)
3. 进入一篇文章
4. 滚动到文章开头，寻找目录折叠区域
5. **点击展开**，验证目录内容正确显示
6. **点击收起**，验证目录正确折叠

预期结果：
- 移动端目录初始状态为**折叠**
- 点击后目录**展开**显示章节列表
- 再次点击**收起**目录

---

### Task 5: 提交变更

**文件：**
- 提交: `source/_config.next.yml`

- [ ] **Step 1: 提交配置变更**

```bash
git add source/_config.next.yml
git commit -m "feat: enable NexT TOC for article navigation"
git push origin main
```

- [ ] **Step 2: 触发 GitHub Actions 部署**

推送后 GitHub Actions 自动部署到 https://kevinchang1001.github.io/shanhe/

---

## 验证清单

- [ ] `source/_config.next.yml` 包含正确的 `toc` 配置
- [ ] `pnpm hexo clean && pnpm build` 无错误
- [ ] 桌面端文章右侧显示目录
- [ ] 目录正确提取 h2、h3 标题
- [ ] 目录显示章节编号
- [ ] 点击目录项跳转到对应章节
- [ ] 滚动时当前章节高亮
- [ ] 移动端目录展开/折叠正常
- [ ] GitHub Pages 部署成功

---

## 依赖项

无新增依赖，使用 NexT v8 内置功能。

## 预计时间

约 15-20 分钟（不含 GitHub 部署等待时间）
