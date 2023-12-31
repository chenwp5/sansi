# Git交互式暂存

当您看到这篇文章的时候，首先需要反思下是出于什么样的目的。如果是出于学习、了解的目的，那么这非常好。如果是由于您工作中经常有大量变更文件想要拆分为多次提交而不是混杂为一次提交，那么您的Git使用习惯就不是很好。当然事无绝对，就有人喜欢这种操作方式，比如我的某位同事。固然他觉得这很有逼格，但是这在本就沉重的工作中，又增加了一些负担，那必然也将伴随着其他风险：

![在这里插入图片描述](./img/yimao.jpg)


## 一、命令简介

Git交互式暂存使我们可以按照自己的意愿选择一个或多个变更文件，甚至是一个文件中的部分变更内容，将那些相关的内容添加到暂存区，以便形成一次提交。

执行命令`git add -i`时，Git进入如下交互界面，以中间空行为分界，我将其分为上下两部分，上部分为工作目录中的变更文件，但不包括新增的文件。下部分为Git交互式暂存支持的命令，如下图所示共8种，而`What now>`后输入命令对应的编号后回车即可执行该命令。

```shell
frost2@frost2-pc MINGW64 /e/Frost2-Files/IdeaWorkSpace/gitdemo/add-demo (master)
$ git add -i
           staged     unstaged path
  1:    unchanged        +1/-1 feature
  2:    unchanged        +1/-1 name
  3:    unchanged        +8/-6 sitemap.json

*** Commands ***
  1: status       2: update       3: revert       4: add untracked
  5: patch        6: diff         7: quit         8: help
What now>
```

例如输入8后回车，出现如下提示：

```shell
frost2@frost2-pc MINGW64 /e/Frost2-Files/IdeaWorkSpace/gitdemo/add-demo (master)
$ git add -i
           staged     unstaged path
  1:    unchanged        +1/-1 feature
  2:    unchanged        +1/-1 name
  3:    unchanged        +8/-6 sitemap.json

*** Commands ***
  1: status       2: update       3: revert       4: add untracked
  5: patch        6: diff         7: quit         8: help
What now> 8
status        - show paths with changes
update        - add working tree state to the staged set of changes
revert        - revert staged set of changes back to the HEAD version
patch         - pick hunks and update selectively
diff          - view diff between HEAD and index
add untracked - add contents of untracked files to the staged set of changes
*** Commands ***
  1: status       2: update       3: revert       4: add untracked
  5: patch        6: diff         7: quit         8: help
What now>
```

命令`help`会给出命令的简介信息：

| 命令          | 说明                                     |
| ------------- | ---------------------------------------- |
| status        | 查看发生了变更的文件                     |
| update        | 选择变更文件将变他们添加到暂存区         |
| revert        | 将已添加的暂存区的文件回退到modified状态 |
| add untracked | 选择Git未跟踪的文件，添加到暂存区        |
| patch         |                                          |
| diff          | 查看暂存区的文件变更内容                 |
| quit          | 退出Git交互式暂存                        |
| help          | 帮助命令                                 |

## 二、update

输入序号2执行命令update，选择需要添加到暂存区的文件，完整操作如下所示。当我们输入2回车后，提示由`What now`变为`Update`，在update后输入我们想要添加到暂存区的文件序号，多个文件用逗号隔开，也可用`1-3`这种格式来选择这个序号范围的文件。选择完成后回车可以看到被选择的文件前会出现`*`,此时不用输入直接回车即可将选中的文件添加到暂存区。

```shell
frost2@frost2-pc MINGW64 /e/Frost2-Files/IdeaWorkSpace/gitdemo/add-demo (master)
$ git add -i
           staged     unstaged path
  1:    unchanged        +1/-1 feature
  2:    unchanged        +1/-1 name
  3:    unchanged        +8/-6 sitemap.json

*** Commands ***
  1: status       2: update       3: revert       4: add untracked
  5: patch        6: diff         7: quit         8: help
What now> 2
           staged     unstaged path
  1:    unchanged        +1/-1 feature
  2:    unchanged        +1/-1 name
  3:    unchanged        +8/-6 sitemap.json
Update>> 1,2
           staged     unstaged path
* 1:    unchanged        +1/-1 feature
* 2:    unchanged        +1/-1 name
  3:    unchanged        +8/-6 sitemap.json
Update>>
updated 2 paths

*** Commands ***
  1: status       2: update       3: revert       4: add untracked
  5: patch        6: diff         7: quit         8: help
What now>
```

## 三、revert

上一步将文件`name`和`feature`添加到了暂存区，输入1查看状态如下所示。

此时如果想要将添加到暂存区的文件回退，可以在`What now`输入3，提示变为`Revert`,同样输入序号选择想要回退的文件，如果选择了没有添加到暂存区的文件，例如本例中的文件`sitemap.json`，那么该文件不会有任务变化，另外两个文件成功退回modified状态。

```shell
*** Commands ***
  1: status       2: update       3: revert       4: add untracked
  5: patch        6: diff         7: quit         8: help
What now> 1
warning: LF will be replaced by CRLF in sitemap.json.
The file will have its original line endings in your working directory
           staged     unstaged path
  1:        +1/-1      nothing feature
  2:        +1/-1      nothing name
  3:    unchanged        +8/-6 sitemap.json

*** Commands ***
  1: status       2: update       3: revert       4: add untracked
  5: patch        6: diff         7: quit         8: help
What now> 3
warning: LF will be replaced by CRLF in sitemap.json.
The file will have its original line endings in your working directory
           staged     unstaged path
  1:        +1/-1      nothing feature
  2:        +1/-1      nothing name
  3:    unchanged        +8/-6 sitemap.json
Revert>> 1,2,3
           staged     unstaged path
* 1:        +1/-1      nothing feature
* 2:        +1/-1      nothing name
* 3:    unchanged        +8/-6 sitemap.json
Revert>>
reverted 3 paths
```

## 四、add untracked

由于上一步中将暂存区的文件都回退了，故目前`git status`状态如下：

```shell
frost2@frost2-pc MINGW64 /e/Frost2-Files/IdeaWorkSpace/gitdemo/add-demo (master)
$ git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   feature
        modified:   name
        modified:   sitemap.json

Untracked files:
  (use "git add <file>..." to include in what will be committed)

        newfile

no changes added to commit (use "git add" and/or "git commit -a")
```

可以看到目前工作目录中有一个新添加的文件`newfile`，但是Git交互式暂存中status不会显示为跟踪的文件，如果需要将它添加到暂存区，需要通过命令`add untracked`，具体步骤如下所示：

```shell
*** Commands ***
  1: status       2: update       3: revert       4: add untracked
  5: patch        6: diff         7: quit         8: help
What now> 4
  1: newfile
Add untracked>> 1
* 1: newfile
Add untracked>>
warning: LF will be replaced by CRLF in newfile.
The file will have its original line endings in your working directory
added 1 path

*** Commands ***
  1: status       2: update       3: revert       4: add untracked
  5: patch        6: diff         7: quit         8: help
What now> 1
           staged     unstaged path
  1:    unchanged        +1/-1 feature
  2:    unchanged        +1/-1 name
  3:        +1/-0      nothing newfile
  4:    unchanged        +8/-6 sitemap.json

*** Commands ***
  1: status       2: update       3: revert       4: add untracked
  5: patch        6: diff         7: quit         8: help
What now>
```

## 五、patch

通过patch命令我们可以将文件中的修改内容分成不用块，然后选择其中一个或多个块将他们添加到暂存区，首先输入5，选择对应的文件后如下所示：

```shell
*** Commands ***
  1: status       2: update       3: revert       4: add untracked
  5: patch        6: diff         7: quit         8: help
What now> 5
           staged     unstaged path
  1:    unchanged        +1/-1 feature
  2:    unchanged        +1/-1 name
  3:    unchanged        +3/-2 suggest.json
Patch update>> 3
           staged     unstaged path
  1:    unchanged        +1/-1 feature
  2:    unchanged        +1/-1 name
* 3:    unchanged        +3/-2 suggest.json
Patch update>>
diff --git a/suggest.json b/suggest.json
index 3caab45..4ba5661 100644
--- a/suggest.json
+++ b/suggest.json
@@ -1,9 +1,10 @@
 {
   "desc": "Git书籍推荐",
-  "book": "Pro Git",
+  "book_name": "Pro Git",
   "book_url": "https://git-scm.com/book/zh/v2",
   "suggest": "git初学者实不建议看各种博客、网站，Git官网书籍《Pro Git》安心读一遍足矣"
 },
 {
-  "author": "frost2"
+  "author": "frost2",
+  "date": "2021-12-30"
 }
Stage this hunk [y,n,q,a,d,s,e,?]?
```

此时提示的`[y,n,q,a,d,s,e,?]`为目前支持的命令，输入`?`可以展示这些命令的说明。如下所示

```shell
Stage this hunk [y,n,q,a,d,s,e,?]? ?
y - stage this hunk
n - do not stage this hunk
q - quit; do not stage this hunk or any of the remaining ones
a - stage this hunk and all later hunks in the file
d - do not stage this hunk or any of the later hunks in the file
s - split the current hunk into smaller hunks
e - manually edit the current hunk
? - print help
@@ -1,9 +1,10 @@
 {
   "desc": "Git书籍推荐",
-  "book": "Pro Git",
+  "book_name": "Pro Git",
   "book_url": "https://git-scm.com/book/zh/v2",
   "suggest": "git初学者实不建议看各种博客、网站，Git官网书籍《Pro Git》安心读一遍足矣"
 },
 {
-  "author": "frost2"
+  "author": "frost2",
+  "date": "2021-12-30"
 }
Stage this hunk [y,n,q,a,d,s,e,?]?
```

此时输入`s`，Git会将修改内容分块，如下所示会显示第一块变更内容，通过`y`和`n`可以选择添加或者不添加到暂存区，本例中我们将第一块变更添加到暂存区，添加后会调到下一块内容，再选择`n`不添加第二块内容到暂存区，具体操作如下所示：

```shell
Stage this hunk [y,n,q,a,d,s,e,?]? s
Split into 2 hunks.
@@ -1,7 +1,7 @@
 {
   "desc": "Git书籍推荐",
-  "book": "Pro Git",
+  "book_name": "Pro Git",
   "book_url": "https://git-scm.com/book/zh/v2",
   "suggest": "git初学者实不建议看各种博客、网站，Git官网书籍《Pro Git》安心读一遍足矣"
 },
 {
Stage this hunk [y,n,q,a,d,j,J,g,/,e,?]? y
@@ -4,6 +4,7 @@
   "book_url": "https://git-scm.com/book/zh/v2",
   "suggest": "git初学者实不建议看各种博客、网站，Git官网书籍《Pro Git》安心读一遍足矣"
 },
 {
-  "author": "frost2"
+  "author": "frost2",
+  "date": "2021-12-30"
 }
Stage this hunk [y,n,q,a,d,K,g,/,e,?]? n

*** Commands ***
  1: status       2: update       3: revert       4: add untracked
  5: patch        6: diff         7: quit         8: help
What now> 1
warning: LF will be replaced by CRLF in feature.
The file will have its original line endings in your working directory
warning: LF will be replaced by CRLF in name.
The file will have its original line endings in your working directory
warning: LF will be replaced by CRLF in suggest.json.
The file will have its original line endings in your working directory
           staged     unstaged path
  1:    unchanged        +1/-1 feature
  2:    unchanged        +1/-1 name
  3:        +1/-0      nothing newfile
  4:        +1/-1        +2/-1 suggest.json

*** Commands ***
  1: status       2: update       3: revert       4: add untracked
  5: patch        6: diff         7: quit         8: help
What now>
```

## 六、diff

命令`diff`可以查看添加到暂存区文件的变更，上一步我们将`suggest.json`的第一块变更添加到了暂存区，正好可以通过命令`diff`验证一下。

```shell
What now> 6
warning: LF will be replaced by CRLF in feature.
The file will have its original line endings in your working directory
warning: LF will be replaced by CRLF in name.
The file will have its original line endings in your working directory
warning: LF will be replaced by CRLF in suggest.json.
The file will have its original line endings in your working directory
           staged     unstaged path
  1:        +1/-0      nothing newfile
  2:        +1/-1        +2/-1 suggest.json
Review diff>> 2
diff --git a/suggest.json b/suggest.json
index 3caab45..39fe4a5 100644
--- a/suggest.json
+++ b/suggest.json
@@ -1,6 +1,6 @@
 {
   "desc": "Git书籍推荐",
-  "book": "Pro Git",
+  "book_name": "Pro Git",
   "book_url": "https://git-scm.com/book/zh/v2",
   "suggest": "git初学者实不建议看各种博客、网站，Git官网书籍《Pro Git》安心读一遍足矣"
 },
*** Commands ***
  1: status       2: update       3: revert       4: add untracked
  5: patch        6: diff         7: quit         8: help
What now>
```

可以看到暂存区中的文件`suggest.json`的变更确为我们添加的第一块内容，输入`7`退出Git交互式暂存，执行`git status`如下所示：

```shell
frost2@frost2-pc MINGW64 /e/Frost2-Files/IdeaWorkSpace/gitdemo/add-demo (master)
$ git status
On branch master
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        new file:   newfile
        modified:   suggest.json

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   feature
        modified:   name
        modified:   suggest.json

```

现假设目前暂存区就是我们想要的一次提交，此时就可以执行`git commit`命令生成一次提交。
