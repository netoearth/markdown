版本控制记录着软件的每一次改变，每一次发布，以及每一个Bug。 它贯穿于软件的生命周期，从生到死，**请慎重对待每一次提交，像记录历史一样书写提交记录**。

当下最流行的Git是一个不错的选择，**作为合格的软件开发人员你应该熟练的使用它**。你可以构造一些场景去练习git命令，比如：

-   在Github创建一个repo，并向其提交代码。（clone, add, commit, push）
-   从main分支创建一个新的分支dev，改变代码，然后通过PR的方式将代码合并到main分支。(checkout, pr, branch)
-   从main分支创建一个新的分支dev，改变代码提交。然后切回main分支，改变代码，提交至远端分支。再切换dev分支，将main分支的提交合并到当前分支。（练习rebase和reset）
-   从main分支创建一个新的分支dev, 创建7个commit，然后将他们合并为1个commit. （练习使用reset或者rebase）
-   从main分支创建一个新的分支dev, 创建7个commit，然后将第七个commit合并到第一个commit，抛弃第二个commit，仅修改第三个commit的commit message，仅修改第四个commit的commit author，修改第六个commit的代码。（练习rebase，以及如何整理commit message）
-   Fork一个repo，本地clone，修改代码。在被fork的repo主分支出现新的提交之后，将本地代码和被fork的repo同步。（练习remote, fetch, rebase）

这里只列举了一些场景，你也可以根据你的需求再写一些场景用来练习。**建议使用命令行练习**，也建议以后的开发过程中使用命令行，最好不要使用IDE提供的UI。

当你了解了Git之后，也需要学习一下常见的git工作流：

-   Git flow
-   Trunk-based
-   Feature branching
-   Forking Workflow

学习资料:

-   官方文档：[Git - Documentation (git-scm.com)](https://git-scm.com/doc). 官方文档已经写的很详尽了。
-   一本书 Pro Git: [https://github.com/progit/progit2/releases/download/2.1.338/progit.pdf](https://github.com/progit/progit2/releases/download/2.1.338/progit.pdf)