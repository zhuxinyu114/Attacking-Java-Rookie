# git

#### github删除提交历史

1. 尝试 运行 git checkout --orphan latest_branch
2. 添加所有文件git add -A
3. 提交更改git commit -am "commit message"
4. 删除分支git branch -D master
5. 将当前分支重命名git branch -m master
6. 最后，强制更新存储库。git push -f origin master

#### 