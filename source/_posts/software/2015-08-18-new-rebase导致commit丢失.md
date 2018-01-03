---
layout: post
title: "rebase导致commit丢失"
description: "rebase操作不当导致commit丢失,rebase操作不当导致提交信息丢失"
category: software
tags: [git, rebase]
---

提交代码合并时,被告知有冲突,同事建议我用rebase来解决,可以保持提交历史的干净.然而我按他的建议进行操作之后,发现我的提交内容丢失了.很郁闷.以下是出错过程:

  > mkdir rebaselost
  > cd rebaselost
  > git init
  > echo "init content" > a
  > git commit -m 'init content'
  > git checkout -b branch1
  > echo "branch1 content" > a
  > git commit -m 'branch1'
  > git checkout master
  > echo "new master content" > a
  > git commit -m 'master content'
  > echo "new master content2" > a
  > git commit -m 'master content2'
  > git checkout branch1
  > git rebase master
  > git rebase --skip

期望的结果是:

>  new master content2  
>  branch1 content   

实际的结果:

>  new master content2

问题出在 rebase里的 --skip 参数,加了这个参数后,原来的提交记录会丢失.

要找回原来的提交记录,可以通过 git reflog 来查看所有的操作记录(包括被删除的commit记录),然后通过git checkout 来返回

> $ git reflog 

操作结果:

```
 fa0f628 HEAD@{0}: rebase: branch1    
 465b616 HEAD@{1}: rebase: checkout master    
 d0350f9 HEAD@{2}: checkout: moving from branch1 to d0350f9  
 7f50104 HEAD@{3}: checkout: moving from master to branch1
 465b616 HEAD@{4}: commit: new master status2  
 7f50104 HEAD@{5}: checkout: moving from d1fe9edd03c54d3b2fdfc0f7a9ea49a7459f5596 to master  
 d1fe9ed HEAD@{6}: rebase: branch1  
 7f50104 HEAD@{7}: rebase: checkout master  
 d0350f9 HEAD@{8}: checkout: moving from branch1 to d0350f9  
 7f50104 HEAD@{9}: rebase finished: returning to refs/heads/branch1  
 7f50104 HEAD@{10}: rebase: checkout master  
 d0350f9 HEAD@{11}: checkout: moving from master to branch1  
 7f50104 HEAD@{12}: commit: master content  
 4fe1cf4 HEAD@{13}: checkout: moving from branch1 to master  
 d0350f9 HEAD@{14}: commit: branch1  
 4fe1cf4 HEAD@{15}: checkout: moving from master to branch1  
 4fe1cf4 HEAD@{16}: commit (initial): init content  
```

> $ git checkout d0350f9


要避免丢失,在冲突时编辑冲突即可


