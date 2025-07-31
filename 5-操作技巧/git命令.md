#### 初始化 [[Git]] 仓库
```bash
git init  # 初始化 Git 仓库
```
---
#### 配置 Git 用户信息
```bash
git config --global user.name "wydyxhxs"  # 设置 GitHub 用户名
git config --global user.email "wydyxhxs@gmail.com"  # 设置 GitHub 邮箱
```
---
#### 创建gitignore文件
```bash
echo ".obsidian/" > .gitignore  # 排除 Obsidian 配置文件夹
echo "*.log" >> .gitignore  # 排除日志文件
echo "*.bak" >> .gitignore  # 排除备份文件
echo "*.swp" >> .gitignore  # 排除编辑器的临时文件
echo "*.jpg" >> .gitignore  # 排除图片文件
echo "*.png" >> .gitignore  # 排除图片文件
```
---
#### 添加文件并提交本地更改
```bash
git add .  # 将所有文件添加到暂存区
git commit -m "Initial commit"  # 提交文件到本地仓库
```
---
#### 创建远程仓库并添加远程仓库地址
```bash
git remote add origin https://github.com/wydyxhxs/CTF.git  # 替换为你的远程仓库地址
```
---
#### 推送本地仓库到 GitHub
```bash
git push -u origin master  # 将更改推送到 GitHub 的 master 分支
```
---
#### 提交和推送更改至Github
```bash
git add .  # 添加所有更改到暂存区
git commit -m "Updated notes"  # 提交更改
git push origin master  # 推送到 GitHub 仓库
```
---
#### 查看仓库状态和历史
```bash
git status  # 查看仓库状态，检查是否有未提交的更改
git log  # 查看提交历史
```
---
#### 回退到以前的版本
```bash
git checkout <commit-id>  # 回退到指定的提交 ID
git reset --hard <commit-id>  # 回退到指定版本并删除后续提交
```
#### 拉取远程仓库的更新
```bash
git pull origin master  # 拉取远程仓库的更新
```
