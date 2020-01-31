# Git
git diff: 比较文件工作区与暂存区的差异
git diff --cached: 比较文件暂存区与上次提交的差异
git commit -a: 跳过git add直接提交，前提是文件必须是追踪的
#### 工作区域
- 非工作区
我个人理解为.gitignore的区域
- 工作区
- 暂存区
- 版本库
#### 文件的几种状态（`git status`）
- Untracked files（未追踪）
不在工作区，任何修改都不会引起git的注意。通过`git add <filename>`追踪
- Tracked-new files（已追踪新文件）
在暂存区等待加入版本库。通过`git rm <filename>`取消追踪。通过`git commit`提交至版本库
- Tracked-modify files（已追踪被修改文件）
工作区或者暂存区的文件与上一版本不一致。
- Tracked-save files(已追踪已保存文件)
工作区的文件与上一版本一致

#### `git add`可以:
- 追踪文件，即将文件添加到工作区。相反操作用`git rm <filename>`
- 将工作区文件提交到暂存区。相反操作用`git reset HEAD <filename>`，实际上是用版本库当前结点覆盖暂存区来还原，这个操作不影响工作区
#### `git commit`：
