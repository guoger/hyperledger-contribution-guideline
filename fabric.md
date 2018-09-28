# Hyperledger Fabric代码贡献
_Hyperledger Fabric（以下简称Fabric）欢迎各位大神积极参与代码的贡献。本文总结并翻译官方文档中的Contribution部分，为大家介绍贡献代码的方法。_

## 基础技能要求
* 基本的命令行使用
* 理解代码版本管理（version control systems），了解git基本操作，例如`git push`，`git commit`
* Docker基础使用，例如`docker ps`，`docker images`等操作
* Go语言基础知识
* 软件自动构建工具make基础知识

## 安装开发环境
* git客户端，通常需要您可以在命令行中执行git命令
* [可选] git review插件,用于支持`git review`命令，安装方法可以参考 https://www.mediawiki.org/wiki/Gerrit/git-review
* Go编程语言1.10.x以及配置好的开发环境，例如GOPATH，GOROOT以及相应的文件夹结构，请参考[Golang文档](https://golang.org/doc)
* Docker 和 Docker Compose
MacOSX, Unix类系统, Windows10需要安装Docker 17.06.2-ce或以上版本。Windows10以下版本的OS需要安装Docker Toolbox，同样要求17.06.2-ce或以上版本。
* 安装libtool
  * Ubuntu: `apt-get install -y libtool`
  * MacOS: `brew install libtool`
* MacOS的用户需要安装以下额外的工具
  * Xcode
  * 执行 `brew install gnu-tar --with-default-names` 安装gnutar来替换自带的bsdtar

## 获取源码
1. 前往https://gerrit.hyperledger.org 并点击右上角Sign in，使用您的LFID(Linux Foundation ID)登陆
    * 如果没有LFID（第一次使用），请前往https://identity.linuxfoundation.org/ 注册一个LFID
2. 在项目列表找到fabric项目，进入项目主页
    * `Projects` -> `List` -> `fabric`
3. 点选左上方的`Clone with commit-msg hook`按钮
    * *错误示例：如果没有登陆你的 LFID，你将看不到 `Clone with commit-msg hook`按钮，只能看到 clone | anonymous http 按钮*
4. 在按钮同一栏的右边，选择验证方式 anonymous http|http|ssh，并复制其下方的命令
    * ssh方式：需要给gerrit配置ssh
      - 参考https://hyperledger-fabric.readthedocs.io/en/latest/Gerrit/lf-account.html#configuring-gerrit-to-use-ssh
      - 缺点：需要先生成本机的ssh pubkey文件，并在https://gerrit.hyperledger.org/ 的个人设置中将pubkey的内容加入列表中，才可以使用。每台开发机都需要做一次
      - 命令形如
      ```
      git clone ssh://<LFID>@gerrit.hyperledger.org:29418/fabric && scp -p -P 29418 <LFID>@gerrit.hyperledger.org:hooks/commit-msg fabric/.git/hooks/
      ```
    * http方式:
      - 运行期间,需要输入LFID对应的密码
      - 命令形如
      ```
      git clone https://<LFID>@gerrit.hyperledger.org/r/a/fabric && (cd fabric && curl -kLo `git rev-parse --git-dir`/hooks/commit-msg https://<LFID>@gerrit.hyperledger.org/r/tools/hooks/commit-msg; chmod +x `git rev-parse --git-dir`/hooks/commit-msg)  
      ```
    * anonymous http方式：与http验证类似
      - 运行期间,需要输入LFID以及对应的密码
      - 命令如下
      ```
      git clone https://gerrit.hyperledger.org/r/fabric && (cd fabric && curl -kLo `git rev-parse --git-dir`/hooks/commit-msg https://gerrit.hyperledger.org/r/tools/hooks/commit-msg; chmod +x `git rev-parse --git-dir`/hooks/commit-msg)
      ```

5. 在$GOPATH/src/github.com/hyperledger下运行复制的命令
    * 无论选用哪种验证方式,复制的命令都会克隆fabric源码，并下载githooks。
6. 下载源码后，前往fabric根目录

## 编译和测试
在fabric/Makefile中，您可以看到许多构建项目。通过make，您可以自动化地编译和测试fabric代码。

**第一次开发前,建议运行`make gotools`安装一些fabric开发依赖的go工具，例如golint**

下面列举几个在代码开发过程中，常用到的命令：
* `make basic-checks`对代码进行一些必要的检查，例如是否有License，拼写错误，代码风格检查等
* `make unit-test`运行所有go的单元测试
* `make integration-test`运行所有的集成测试
* `make peer`/`make orderer`编译peer或orderer的二进制文件，可以在`.build/bin`中找到
* `make docker`或`make peer-docker`/`make orderer-docker`编译所有docker镜像，或指定的docker镜像。该镜像会被打上latest标签。在手工启动docker容器时，您可以使用这些新编译的镜像来测试您的代码
* `make clean`删除所有的编译文件
这里仅仅列出了一些您可能在开发中常用的命令。我们强烈建议您浏览Makefile最顶部的注释部分，这有助于您了解Fabric的编译系统。这对您以后的开发极为有益。通常来讲，在您修改了fabric代码之后，应当运行`make basic-checks`和`make unit-test`来避免显而易见的错误。

## 修改代码
在提交代码之前，您应当明确您的修改所对应的JIRA Issue。通常来讲，您应当将对应的Issue assign给您自己，避免别人重复您的工作。

通常来讲，您的修改应当在master分支上进行。
当您修改完成后，使用`git commit -s`产生一个新的commit，并使用如下模板：

  > [FAB-XXXX] <标题,不能超过55个字符>  
  > <空行>  
  > <内容：每行不超过80字符，其中应该包括  
  >     - 你的提交做了什么？  
  >     - 为何选用这种方式去进行改动  
  >     - 改动的正确性，比如提交你成功的代码测试结果
  >     
  > >
_tips: 当您成功创建commit之后，可以使用`git log`查看commit message。您会看到其中除了您刚才填写的内容外，githook还自动为您添加了change-id。在您之后更新修改并再次提交时，gerrit会根据此id确定是否创建新的Change Request(CR)或更新已有的_

## 提交代码
使用如下命令提交代码：
```
git push origin HEAD:refs/for/master
```
如果运行成功，您将会看到如下信息：
```
Counting objects: 3, done.
Writing objects: 100% (3/3), 306 bytes | 0 bytes/s, done.
Total 3 (delta 0), reused 0 (delta 0)
remote: Processing changes: new: 1, refs: 1, done
remote:
remote: New Changes:
remote:   https://gerrit.hyperledger.org/r/6 Test commit // 您可以使用链接查看您的提交
remote:
To ssh://LFID@gerrit.hyperledger.org:29418/fabric
* [new branch]      HEAD -> refs/for/master
```

_tips：您还可以使用`git review`来提交代码，请参考[这里](https://hyperledger-fabric.readthedocs.io/en/latest/Gerrit/gerrit.html#using-git-review)_

## 修改您的提交
当您在gerrit中收到反馈后，您可以修改您的提交。此时，请使用`git commit --amend`来复写您的commit，并仍然使用`git push`或者`git review`来提交。当commit中的change-id保持不变时，gerrit会自动在已有的CR基础上进行更新。如果您由于某些原因创建了新的commit，请用原有的change-id覆盖新生成的，以免创建新的CR。

## 完整流程
_此处插入完整流程图_

## FAQ

### Q: 我很有兴趣贡献代码，但是不知道应该从哪里开始？
在初入一个开源社区时，如何下手是最常见的问题。通常来讲，您需要从小事做起，比如修复一些bug，修改和编撰文档，而建立与社区的联系与信任。这里有几点建议：
- 搜索标签`help-wanted`，挑选还未被assign的工作来进行。
- 使用fabric进行开发，并找出使用过程中遇到的问题，尝试在源码中寻找问题的源头，如果认为是bug，提交一个JIRA并尝试修复。如果认为是开发体验需要改善，也可以提交JIRA并参与讨论。
- 参加社区的会议，无论是在Rocket.chat还是邮件列表，您都可以听到来自全球许多开发者的声音。这有助于您了解最新的动向，并找到您擅长或感兴趣的贡献方向。
- 代码并不是唯一的贡献，开源社区也远不止代码。在社区中帮助回答或指出一个问题，修改文档中的错误或过时的信息，都是贡献的一种。

### Q: 我提交了代码之后，应该找谁来review？
通常来讲，您可以使用`git blame`来找到您修改的部分由谁维护，并添加TA为reviewer。请一定仔细阅读文档中[关于代码风格的段落](https://hyperledger-fabric.readthedocs.io/en/latest/CONTRIBUTING.html#what-makes-a-good-change-request)。这有助于您的修改被顺利提交。一些值得注意的事项：
- 请不要在一个提交中完成所有的事情，使得一个commit包含太多改动。这在几乎所有的开源社区中，都不是一个好的习惯。如果改动过多，请分解成几个逻辑独立的commit。**尽量使得每个提交不超过500行修改**。您可以从reviewer的角度换位思考，您希望看到怎样的代码风格。
- 您的修改需要被测试代码覆盖。
- 社区的reviewer通常来讲都比较繁忙，请有一些耐心。如果您的代码不涉及严重的安全漏洞，可能优先级不会很高，请理解。

### Q: 为什么我的代码被merge了之后，在github的contributors页面里没有显示？
Fabric的页面默认显示release分支上的数据，而您的代码通常来讲被merge到了master分支。在您的代码被发布到一个release版本中后，您会看到您的信息显示在contributor页面里。


