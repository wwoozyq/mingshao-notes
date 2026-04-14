# 关于这个站点

这个模板基于 `MkDocs + Material for MkDocs`，现在已经按你的写作方向整理成了一个以 `AI`、`数学基础`、`港新留学`、`计算机刷题与基础` 为主的个人知识库。

## 当前已经配置好的内容

- 中文界面
- 按 AI、数学基础、港新留学、计算机拆分的栏目导航
- 站内搜索
- 代码复制按钮
- 适合首页展示的基础样式

## 目前的推荐目录

- `docs/ai/`：学习路线、模式识别与机器学习、自然语言处理
- `docs/math/`：线性代数、概率统计、基础公式与推导
- `docs/study-abroad/`：港新选校、申请规划、考试、文书、签证
- `docs/computer/`：数据结构基础、Hot100 刷题、题解总结
- `docs/journal/`：按时间记录周记、进度和阶段总结

## 你之后最常改的地方

### 1. 新增页面

把新的 Markdown 文件放进 `docs/` 目录，然后在 `mkdocs.yml` 里把它加到导航中。

### 2. 改站点信息

这些配置都在 `mkdocs.yml`：

- `site_name`
- `site_description`
- `site_author`
- `site_url`

### 3. 改首页风格

如果你想调整颜色、标题区块或卡片样式，可以修改 `docs/stylesheets/extra.css`。

## 本地运行

```bash
mkdocs serve
```

## 生成静态文件

```bash
mkdocs build
```

构建完成后，静态站点会输出到 `site/` 目录。
