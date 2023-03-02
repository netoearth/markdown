这篇文章将讨论如何将 Git 存储库克隆到特定文件夹中。

克隆的标准方法是存储库使用 [git-clone](https://git-scm.com/docs/git-clone) 命令。但是当你简单地克隆一个存储库时 `git clone <repository>`，它会在文件系统的当前路径中创建一个具有存储库名称的新目录，并在其中克隆存储库。

如果要将远程存储库克隆到特定目录，可以指定其路径：

$ git clone <repository> <path>

这里， `<path>` 是要克隆到的目录的路径。这在下面进行了演示，其中克隆到现有的本地目录中。

![git clone directory](https://www.techiedelight.com/wp-content/uploads/git-clone-directory.png)

   
如果要将 git 存储库克隆到当前目录，可以这样做：

$ git clone <repository> .

在这里，点 `(.)` 表示当前目录。

![git clone .](https://www.techiedelight.com/wp-content/uploads/git-clone-..png)

   
或者，您可以使用 [git \[-C \]](https://git-scm.com/docs/git#Documentation/git.txt--Cltpathgt) 选项来指定根目录。这是因为 git 会假设 `<path>` 作为当前工作目录。

$ git -C <path> clone <repository>

请注意，这会在指定目录中创建一个具有存储库名称的新目录，并在其中克隆存储库，如下所示：

![git -C directory clone](https://www.techiedelight.com/wp-content/uploads/git-C-directory-clone.png)

   
但是，您只能在现有目录为空时克隆到现有目录。否则，git 会报错目标路径已经存在且不是空目录。

![git error – not an empty directory](https://www.techiedelight.com/wp-content/uploads/git-errror-not-an-empty-directory.png)

   
如果目的地不为空，您可以这样做：

\# create and initialize an empty repository  
$ git init

\# add a remote named origin for the repository at <repository>  
$ git remote add origin <repository>

\# do a git-fetch  
$ git fetch

\# check out the master branch  
$ git checkout master

这种方法如下所示：

![git fetch](https://www.techiedelight.com/wp-content/uploads/git-init-fetch.png)

这就是将 Git 存储库克隆到特定文件夹中的全部内容。

  

**谢谢阅读。**

请使用我们的 [在线编译器](https://www.techiedelight.com/zh/compiler/) 使用 C、C++、Java、Python、JavaScript、C#、PHP 和许多更流行的编程语言在评论中发布代码。

**像我们？将我们推荐给您的朋友，帮助我们成长。快乐编码** 🙂