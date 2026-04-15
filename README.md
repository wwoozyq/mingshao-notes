# 茗苕芝士铺

一个基于 `MkDocs + Material for MkDocs` 的个人知识网站，当前内容结构聚焦：

- AI：模式识别与机器学习、自然语言处理
- 生物医学：生物医学图像处理
- 留学：香港 / 新加坡申请
- 计算机：数据结构基础、LeetCode Hot100

## 本地启动

```bash
source .venv/bin/activate
mkdocs serve
```

## 本地构建

```bash
source .venv/bin/activate
mkdocs build
```

## 上传到 GitHub

如果你还没初始化仓库，可以直接在项目根目录运行：

```bash
git init
git add .
git commit -m "Initial site"
git branch -M main
git remote add origin https://github.com/<你的用户名>/<仓库名>.git
git push -u origin main
```

## GitHub Pages 自动发布

这个项目已经包含 GitHub Actions 工作流文件 `.github/workflows/deploy.yml`。

推到 GitHub 之后：

1. 打开仓库的 `Settings` -> `Pages`
2. 在 `Build and deployment` 里选择 `GitHub Actions`
3. 确保默认分支是 `main`
4. 之后每次推送到 `main` 都会自动构建并发布

## 绑定自定义域名

当前项目的正式站点域名已经设置为 `https://mingshaoai.xyz`。

GitHub Pages 绑定域名时，常见做法有两种：

- 根域名，例如 `mingshaoai.xyz`
- 子域名，例如 `notes.mingshaoai.xyz`

根域名通常需要配置 `A` 记录；子域名通常配置 `CNAME` 记录。域名验证与 HTTPS 建议在 GitHub Pages 后台一起完成。
