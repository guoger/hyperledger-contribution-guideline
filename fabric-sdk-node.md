# Hyperledger Fabric Nodejs SDK代码贡献指南
Fabric nodejs sdk, 又称fabric-sdk-node， 是基于nodejs的Fabric后端开发工具集。

它是fabric各个sdk中版本发布较齐备，代码迭代较快速的一个。同时也是较早发布的sdk之一。

[项目gerrit主页面链接](https://gerrit.hyperledger.org/r/#/admin/projects/fabric-sdk-node)

[项目源码github镜像](https://github.com/hyperledger/fabric-sdk-node)

本文总结并翻译官方文档当中与代码贡献有关的部分

# 基础技能要求

- gulpjs 基本命令使用
 *gulp 是nodejs语境下的自动化构建工具框架,类似java下的maven,gradle*
- nodejs 编程

# 开发环境

- Nodejs 8.x, **Node v9.0+ 暂时不支持**
- NPM 5.5.1 或更高版本
- gulp 命令工具 
  - 安装命令:`npm install -g gulp`
- docker 和 docker-compose: 用于集成测试
  - 版本与Fabric源码开发所需的环境相同
- (可选) Fabric相关的docker镜像
  - 某些镜像在构建过程中可以自动下载
  
# 获取源码
1. 前往https://gerrit.hyperledger.org 并点击右上角Sign in，使用您的LFID(Linux Foundation ID)登陆
    * 如果没有LFID（第一次使用），请前往https://identity.linuxfoundation.org/ 注册一个LFID
2. 在项目列表找到fabric项目，进入项目主页
    * `Projects` -> `List` -> `fabric-sdk-node`
3. 点选左上方的`Clone with commit-msg hook`按钮
    * *错误示例：如果没有登陆你的 LFID，你将看不到 `Clone with commit-msg hook`按钮，只能看到 clone | anonymous http 按钮*
4. 在按钮同一栏的右边，选择验证方式 anonymous http|http|ssh，并复制其下方的命令
    * ssh方式：需要给gerrit配置ssh
      - 参考https://hyperledger-fabric.readthedocs.io/en/latest/Gerrit/lf-account.html#configuring-gerrit-to-use-ssh
      - 缺点：需要先生成ssh keypair,将其加入本机的密钥管理器中，并在https://gerrit.hyperledger.org/ 的个人设置中将pubkey的内容加入列表中，才可以使用。因此每台开发机都需要做一次
      - 命令形如
      ```
      git clone ssh://<LFID>@gerrit.hyperledger.org:29418/fabric-sdk-node && scp -p -P 29418 <LFID>@gerrit.hyperledger.org:hooks/commit-msg fabric-sdk-node/.git/hooks/
      ```
    * http方式:
      - 运行期间,需要输入LFID对应的密码
      - 命令形如
      ```
      git clone https://<LFID>@gerrit.hyperledger.org/r/a/fabric-sdk-node && (cd fabric-sdk-node && curl -kLo `git rev-parse --git-dir`/hooks/commit-msg https://<LFID>@gerrit.hyperledger.org/r/tools/hooks/commit-msg; chmod +x `git rev-parse --git-dir`/hooks/commit-msg)
      ```
    * anonymous http方式：与http验证类似
      - 运行期间,需要输入LFID以及对应的密码
      - 命令如下
      ```
      git clone https://gerrit.hyperledger.org/r/fabric-sdk-node && (cd fabric-sdk-node && curl -kLo `git rev-parse --git-dir`/hooks/commit-msg https://gerrit.hyperledger.org/r/tools/hooks/commit-msg; chmod +x `git rev-parse --git-dir`/hooks/commit-msg)
      ```

5. 在任意目录下运行复制的命令
    * 无论选用哪种验证方式,复制的命令都会克隆fabric-sdk-node源码，并下载githooks。
6. 下载源码后，前往fabric-sdk-node根目录 `$ cd fabric-sdk-node`

# 编译与测试

1. 安装npm依赖  `$ npm install`
2. 运行代码字面测试  `$ gulp` , 字面测试包括
    - ESlint代码格式测试
        - *ESlint是一套javascript代码风格规范工具, 参见  https://eslint.org/*
    - 代码许可标签(license)测试
3. 运行单元测试 `$ gulp test-headless` 或 `$ npm test`
4. 运行整体测试 `$ gulp test`
  - 整体测试涵盖了以上的字面测试和单元测试,此外还包括了大量的集成测试,总测试用例数量2000+
  - **整体测试总耗时估计将超过20分钟**
  
# 写在在修改代码之前
- *贡献者守则1: 代码更改的目的,是为了修复jira上的issue的.即便更改是为了加入新功能，也要创造新issue去'修复'. 换言之,每次更改都必须对应一个issue号*

- *贡献者守则2: 在一个更改当中,变动的代码行数原则上不应该超过500行,否则审阅者很难作出评估.如果确实需要超过500行变动去完成一个功能,则建议开发者将它分拆成几个更改来完成. 对于开发者来说,需要仔细考虑这些子更改的可测试性.如果子更改无法通过Jobbuilder自动化测试,则通常不会被审阅*

- *贡献者守则3: "提交信息模板"*

  > [FABN-XXXX] <标题,不能超过55个字符>  
  > <空行>  
  > <内容：每行不超过80字符，其中应该包括  
  >     - 你的提交做了什么？  
  >     - 为何选用这种方式去进行改动  
  >     - 改动的正确性，比如提交你成功的代码测试结果
  >     
  > >
- 提醒1: Hyperledger的JIRA issue结构已更新,现在Fabric-sdk-node已经被分离出来成为独立的的JIRA项目,缩写为 **FABN**

# 可选流程1: 使用git review插件
0. 安装git review插件
安装方式参见: https://www.mediawiki.org/wiki/Gerrit/git-review#Installation
比如:
- Debian/Ubuntu: ` sudo apt-get install git-review`  
- MacOS: `brew install git-review`
1. 重命名远端: `$ git remote rename origin gerrit`
- **如果之前获取源码时候选择的是 ssh方式,则可以跳过此步**
2. 创建新分支  `$ git checkout -b FABN-XXXX`
3. 修改代码后,创建提交,并遵循 *提交信息模板* 来撰写提交信息
  `$ git commit -s -a`
  - `-a` 为可选项, 具体含义请参见git手册
4. 推送更改: `$ git review`

# 可选流程2: 不使用git review插件

参见 Fabric核心贡献指南里的相关章节
- [修改代码](./fabric.md#%E4%BF%AE%E6%94%B9%E4%BB%A3%E7%A0%81)
- [提交代码](./fabric.md#%E6%8F%90%E4%BA%A4%E4%BB%A3%E7%A0%81)

# 附录

代码结构与技术选型
--------------
- 构建工具： gulpjs框架
- 默认logger实现： winstonjs
- 测试框架
  - tape 第一版的测试框架(现在正逐步迁移到cucumber测试框架)
  - mocha 用于单元测试
- npm发布模组
  - fabric-client 主模组，包含绝大多数sdk的内容
  - fabric-ca-client 可选模组， 仅用于与fabric-ca服务交互


活跃维护者列表
-------------
此表仅只列出回复频率较高且专注于node-sdk的部分维护者

| 姓名                      | Gerrit ID           | GitHub ID         | RocketChat ID  | email                               |
| ------------------------- | ------------------- | ---------------- | -------------- | ----------------------------------- |
| Bret Harrison             | bretharrison        | harrisob         | bretharrison   | beharrison@nc.rr.com                |
| Chaoyi Zhao               | zhaochy             | zhaochy1990      | zhaochy        | zhaochy_2015@hotmail.com            |
| Andrew Coleman            | andrew-coleman      | andrew-coleman   | andrew-coleman | andrew_coleman@uk.ibm.com           |





