# Git 常用命令速查表

## 1. 基础配置

```bash
# 查看 Git 版本
git --version

# 设置用户名
git config --global user.name "Your Name"

# 设置邮箱
git config --global user.email "your@email.com"

# 查看全部配置
git config --list

# 查看用户名
git config user.name

# 查看邮箱
git config user.email
```

---

## 2. 创建与获取仓库

```bash
# 初始化当前目录为 Git 仓库
git init

# 克隆远程仓库
git clone <仓库地址>

# 指定本地目录名
git clone <仓库地址> <目录名>
```

示例：

```bash
git clone https://github.com/user/project.git
git clone https://github.com/user/project.git my-project
```

---

## 3. 查看仓库状态

```bash
# 查看当前状态
git status

# 简洁显示
git status -s
```

常见状态：

|标记|含义|
|---|---|
|`M`|文件已修改|
|`A`|新增文件|
|`D`|文件已删除|
|`??`|未跟踪文件|

---

## 4. 暂存文件

```bash
# 暂存指定文件
git add <文件名>

# 暂存多个文件
git add file1 file2 file3

# 暂存当前目录全部修改
git add .

# 暂存所有修改
git add -A
```

常用：

```bash
git add .
```

---

## 5. 提交代码

```bash
# 提交暂存区内容
git commit -m "提交说明"
```

示例：

```bash
git commit -m "feat: 添加登录功能"
git commit -m "fix: 修复登录异常"
git commit -m "docs: 更新 README"
```

常见流程：

```bash
git status
git add .
git commit -m "feat: 完成用户登录"
```

---

## 6. 查看提交历史

```bash
# 查看详细日志
git log

# 单行显示
git log --oneline

# 图形化显示全部分支
git log --oneline --graph --all

# 更完整的图形化显示
git log --oneline --graph --decorate --all
```

示例：

```text
82ac35f fix: 修复登录错误
7c28b51 feat: 添加登录页面
afe1298 init: 初始化项目
```

前面的字符串是 Commit ID：

```text
82ac35f
```

---

## 7. 查看代码修改

```bash
# 查看工作区未暂存的修改
git diff

# 查看已经暂存的修改
git diff --staged

# 等价写法
git diff --cached

# 比较两个提交
git diff <commit1> <commit2>
```

---

# 分支操作

## 8. 查看分支

```bash
# 查看本地分支
git branch

# 查看远程分支
git branch -r

# 查看所有分支
git branch -a
```

---

## 9. 创建与切换分支

推荐使用 `git switch`：

```bash
# 切换分支
git switch <分支名>

# 创建并切换分支
git switch -c <分支名>
```

示例：

```bash
git switch main
git switch -c feature/login
```

传统写法：

```bash
git checkout <分支名>
git checkout -b <分支名>
```

---

## 10. 删除分支

```bash
# 删除已合并的本地分支
git branch -d <分支名>

# 强制删除本地分支
git branch -D <分支名>

# 删除远程分支
git push origin --delete <分支名>
```

---

## 11. 重命名分支

```bash
# 重命名当前分支
git branch -m <新分支名>
```

示例：

```bash
git branch -m main
```

---

## 12. 合并分支

```bash
git switch <目标分支>
git merge <需要合并的分支>
```

例如把 `dev` 合并到 `main`：

```bash
git switch main
git merge dev
```

> `git merge dev` 表示：把 `dev` 合并到当前分支。

---

# 远程仓库

## 13. 查看远程仓库

```bash
# 查看远程仓库名称
git remote

# 查看远程仓库详细地址
git remote -v
```

通常默认远程仓库名：

```text
origin
```

---

## 14. 添加与修改远程仓库

```bash
# 添加远程仓库
git remote add origin <仓库地址>

# 修改远程仓库地址
git remote set-url origin <新地址>

# 删除远程仓库
git remote remove origin
```

---

## 15. 推送代码

```bash
# 首次推送并关联远程分支
git push -u origin main

# 后续推送
git push

# 指定仓库和分支
git push origin main
```

常见：

```bash
git add .
git commit -m "feat: 添加登录功能"
git push
```

---

## 16. 拉取代码

```bash
# 拉取并合并远程更新
git pull

# 指定远程仓库和分支
git pull origin main
```

`git pull` 可以理解为：

```text
git fetch
+
git merge
```

---

## 17. 获取远程更新

```bash
# 获取远程仓库最新信息，但不自动合并
git fetch

# 指定远程仓库
git fetch origin
```

区别：

|命令|作用|
|---|---|
|`git fetch`|下载远程更新，不自动合并|
|`git pull`|下载远程更新，并尝试合并|

---

# 撤销与回退

## 18. 撤销工作区修改

```bash
# 撤销指定文件修改
git restore <文件名>

# 撤销全部未暂存修改
git restore .
```

注意：未提交的修改可能丢失。

---

## 19. 取消暂存

```bash
# 取消指定文件暂存
git restore --staged <文件名>

# 取消全部暂存
git restore --staged .
```

代码修改仍然保留。

---

## 20. 修改最近一次提交

修改 Commit 信息：

```bash
git commit --amend -m "新的提交说明"
```

漏提交文件：

```bash
git add <遗漏文件>
git commit --amend
```

已经推送到多人协作仓库的 Commit，谨慎使用 `--amend`。

---

## 21. reset 回退提交

### 保留修改并保留暂存状态

```bash
git reset --soft HEAD~1
```

效果：

```text
删除最近一次 Commit
代码保留
暂存状态保留
```

### 保留代码，取消暂存

```bash
git reset HEAD~1
```

等价于：

```bash
git reset --mixed HEAD~1
```

效果：

```text
删除最近一次 Commit
代码保留
取消暂存
```

### 强制回退

```bash
git reset --hard HEAD~1
```

效果：

```text
删除 Commit
删除代码修改
```

谨慎使用。

---

## 22. HEAD 含义

```text
HEAD     当前提交
HEAD~1   上一个提交
HEAD~2   上上个提交
```

示例：

```bash
git reset --soft HEAD~1
```

表示回退一个 Commit。

---

## 23. revert 撤销提交

```bash
git revert <commitID>
```

示例：

```bash
git revert 82ac35f
```

特点：

- 不删除原 Commit
    
- 会创建一个新的“撤销 Commit”
    
- 适合已经推送到远程仓库的提交
    
- 多人协作通常优先考虑 `revert`
    

对比：

|命令|特点|
|---|---|
|`reset`|修改提交历史|
|`revert`|新建提交撤销旧提交|

---

# 临时保存

## 24. stash

代码写到一半，需要临时切换分支：

```bash
git stash
```

恢复：

```bash
git stash pop
```

常用命令：

```bash
# 查看 stash
git stash list

# 保存并添加说明
git stash push -m "登录功能开发到一半"

# 恢复，但保留 stash
git stash apply

# 恢复并删除 stash
git stash pop

# 删除最近一个 stash
git stash drop

# 删除全部 stash
git stash clear
```

典型场景：

```bash
git stash
git switch main

# 处理其他任务

git switch dev
git stash pop
```

---

# 文件操作

## 25. 删除文件

```bash
git rm <文件名>
git commit -m "删除文件"
```

也可以：

```bash
rm <文件名>
git add .
git commit -m "删除文件"
```

---

## 26. 重命名文件

```bash
git mv <旧文件名> <新文件名>
```

示例：

```bash
git mv old.txt new.txt
```

---

# .gitignore

## 27. 忽略文件

`.gitignore` 用来指定不需要 Git 管理的文件。

### 通用

```gitignore
.DS_Store
Thumbs.db
*.log
.env
```

### IDE

```gitignore
.vscode/
.idea/
*.iml
```

### C / C++

```gitignore
*.exe
*.o
*.obj
build/
```

### Java

```gitignore
*.class
target/
```

### Python

```gitignore
__pycache__/
*.pyc
venv/
.env
```

### Node.js

```gitignore
node_modules/
dist/
.env
```

---

# 历史与排查

## 28. 查看文件历史

```bash
# 查看文件提交历史
git log <文件名>

# 查看文件每次具体修改
git log -p <文件名>
```

---

## 29. 查看某行代码是谁修改的

```bash
git blame <文件名>
```

示例：

```bash
git blame main.c
```

---

## 30. reflog 找回误删提交

```bash
git reflog
```

例如误执行：

```bash
git reset --hard HEAD~3
```

先查：

```bash
git reflog
```

找到之前的 Commit：

```text
82ac35f HEAD@{1}: commit: feat: 登录功能
```

恢复：

```bash
git reset --hard 82ac35f
```

`git reflog` 是处理 Git 误操作时非常重要的命令。

---

# 标签

## 31. tag

```bash
# 查看所有标签
git tag

# 创建标签
git tag v1.0

# 给指定 Commit 创建标签
git tag v1.0 <commitID>

# 推送指定标签
git push origin v1.0

# 推送全部标签
git push origin --tags
```

---

# 高级操作

## 32. cherry-pick

把指定 Commit 应用到当前分支：

```bash
git cherry-pick <commitID>
```

示例：

```bash
git switch main
git cherry-pick 82ac35f
```

适合只需要另一个分支中的某一个或几个 Commit。

---

## 33. rebase

```bash
git rebase <目标分支>
```

例如：

```bash
git switch feature/login
git rebase main
```

发生冲突：

```bash
git add .
git rebase --continue
```

取消：

```bash
git rebase --abort
```

用途：

- 整理提交历史
    
- 保持提交历史更线性
    
- 将当前分支的 Commit 移动到最新基线上
    

多人协作时，已经推送并被其他人使用的 Commit 不宜随意 Rebase。

---

## 34. 清理未跟踪文件

先预览：

```bash
git clean -n
```

删除未跟踪文件：

```bash
git clean -f
```

连目录一起删除：

```bash
git clean -fd
```

建议始终先：

```bash
git clean -n
```

---

# 冲突处理

## 35. Merge 冲突

发生冲突时，文件中可能出现：

```text
<<<<<<< HEAD
当前分支代码
=======
另一个分支代码
>>>>>>> dev
```

手动修改为最终代码，并删除冲突标记。

然后：

```bash
git add .
git commit
```

查看冲突状态：

```bash
git status
```

---

# 常用工作流

## 36. 个人项目

```bash
git status
git add .
git commit -m "feat: 完成 xxx 功能"
git push
```

---

## 37. 多人协作

```bash
git pull
git status
git add .
git commit -m "feat: 完成 xxx 功能"
git push
```

更稳妥的方式：

```bash
git fetch
git status
```

确认远程变化后再决定如何合并。

---

## 38. Feature 分支开发

```bash
# 更新主分支
git switch main
git pull

# 创建功能分支
git switch -c feature/login

# 开发并提交
git add .
git commit -m "feat: 实现登录功能"

# 更新 main
git switch main
git pull

# 合并功能分支
git merge feature/login

# 推送
git push

# 删除功能分支
git branch -d feature/login
```

---

# 常用命令速查

## 39. 高频命令

```bash
git status

git add .
git commit -m "message"

git pull
git push

git log --oneline

git diff

git branch
git switch <branch>
git switch -c <branch>

git merge <branch>

git stash
git stash pop

git restore <file>
git restore --staged <file>

git reset --soft HEAD~1
git revert <commit>

git reflog
```

---

# Git 核心流程

## 40. 本地提交流程

```text
工作区
  │
  │ git add
  ▼
暂存区
  │
  │ git commit
  ▼
本地仓库
  │
  │ git push
  ▼
远程仓库
```

远程更新：

```text
远程仓库
  │
  ├── git fetch ──> 获取更新，不自动合并
  │
  └── git pull  ──> 获取更新并合并
```

---

# 41. Git 命令选择表

|需求|命令|
|---|---|
|查看当前状态|`git status`|
|添加全部修改|`git add .`|
|提交代码|`git commit -m "message"`|
|查看历史|`git log --oneline`|
|查看修改|`git diff`|
|查看分支|`git branch`|
|切换分支|`git switch branch`|
|新建分支|`git switch -c branch`|
|合并分支|`git merge branch`|
|拉取代码|`git pull`|
|获取远程更新|`git fetch`|
|推送代码|`git push`|
|临时保存修改|`git stash`|
|恢复临时修改|`git stash pop`|
|撤销未暂存修改|`git restore file`|
|取消暂存|`git restore --staged file`|
|撤销已发布 Commit|`git revert commit`|
|回退本地 Commit|`git reset`|
|找回误删 Commit|`git reflog`|
|应用单个 Commit|`git cherry-pick commit`|

---

# 42. 容易混淆的命令

## `git fetch` vs `git pull`

```text
fetch = 获取远程更新
pull  = 获取远程更新 + 合并
```

---

## `git restore` vs `git reset`

```text
restore
主要处理文件修改、暂存状态

reset
主要处理 Commit、HEAD 和暂存区
```

---

## `git reset` vs `git revert`

```text
reset
修改历史，适合本地未共享的 Commit

revert
创建新的撤销 Commit，适合已经共享的 Commit
```

---

## `git merge` vs `git rebase`

```text
merge
保留分支合并关系
操作相对直观

rebase
重新排列提交历史
历史更线性
```

---

## `git stash apply` vs `git stash pop`

```text
apply
恢复 stash
stash 记录继续保留

pop
恢复 stash
成功后删除对应 stash
```

---

# 43. 出问题时优先执行

不知道 Git 当前发生了什么：

```bash
git status
```

不知道提交历史：

```bash
git log --oneline --graph --decorate --all
```

不知道自己刚才做了什么：

```bash
git reflog
```

担心 `git clean` 删除错误：

```bash
git clean -n
```

需要查看修改：

```bash
git diff
git diff --staged
```

需要取消正在进行的 Rebase：

```bash
git rebase --abort
```

---

# 44. 最小必背集合

```bash
git status
git add .
git commit -m "message"

git pull
git push

git log --oneline
git diff

git branch
git switch
git switch -c
git merge

git stash
git stash pop

git restore
git restore --staged

git reset
git revert
git reflog
```

其中最核心：

```bash
git status
git add .
git commit -m "message"
git pull
git push
```