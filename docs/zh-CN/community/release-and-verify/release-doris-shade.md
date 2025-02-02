---
{
"title": "发布 Doris Shade",
"language": "zh-CN"
}
---

<!-- 
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

# 发布 Doris Shade

其代码库独立于 Doris 主代码库，位于：

- https://github.com/apache/doris-Shade

## 准备发布

首先，请参阅 [发版准备](./release-prepare.md) 文档进行发版准备。

## 发布到 Maven

我们以发布 Doris Shade v1.0.0 为例。

### 1. 准备分支

在代码库中创建分支：1.0.0-release，并 checkout 到该分支。

### 2. 发布到 Maven staging

执行以下命令开始生成 release tag：

```bash
mvn release:clean
mvn release:prepare -DpushChanges=false
```

其中 `-DpushChanges=false` 表示执行过程中，不会向代码库推送新生成的分支和 tag。

在执行 `release:prepare` 命令后，会要求提供以下三个信息：

1. Doris Shade 的版本信息：我们默认就可以，可以直接回车或者输入自己想要的版本。版本格式为 `{shade.version}`，如 `1.0.0`。
2. Doris Shade 的 release tag：release 过程会在本地生成一个 tag。我们使用默认的 tag 名称即可，如 `1.0.0`。
3. Doris Shade 下一个版本的版本号：这个版本号只是用于生成本地分支时使用，无实际意义。我们按规则填写一个即可，比如当前要发布的版本是：`1.0.0`，那么下一个版本号填写 `1.0.1-SNAPSHOT` 即可。

`mvn release:prepare` 可能会要求输入 GPG passphrase。如果出现 `gpg: no valid OpenPGP data found` 错误，则可以执行 `export GPG_TTY=$(tty)` 后在尝试。

`mvn release:prepare` 执行成功后，会在本地生成一个 tag 和一个 branch。并且当前分支会新增两个 commit。第一个 commit 对应的是新生成的 tag，第二个则是下一个版本的 branch。可以通过 `git log` 查看。

本地 tag 确认无误后，需要将 tag 推送到代码库：

`git push upstream --tags`

其中 upstream 指向 `apache/doris-shade` 代码库。

最后，执行 perform:

```
mvn release:perform
```

执行成功后，在 [https://repository.apache.org/#stagingRepositories](https://repository.apache.org/#stagingRepositories) 里面可以找到刚刚发布的版本：

![](/docs/images/staging-repositories.png)

**注意需要包含 `.asc` 签名文件。**

如果操作有误。需要将本地 tag，代码库中的 tag 以及本地新生成的两个 commit 删除。并将 staging drop 掉。然后重新执行上述步骤。

检查完毕后，点击图中的 `close` 按钮完成 staging 发布。

### 3. 准备 svn

检出 svn 仓库：

```
svn co https://dist.apache.org/repos/dist/dev/doris/
```

打包 tag 源码，并生成签名文件和sha256校验文件。这里我们以 `1.0.0` 为例。其他 tag 操作相同

```
git archive --format=tar 1.14_2.12-1.0.0 --prefix=apache-doris-shade-1.0.0-src/ | gzip > apache-doris-shade-1.0.0-src.tar.gz
gpg -u xxx@apache.org --armor --output apache-doris-shade-1.0.0-src.tar.gz.asc  --detach-sign apache-doris-shade-1.0.0-src.tar.gz
sha512sum apache-doris-shade-1.14_2.12-1.0.0-src.tar.gz > apache-doris-shade-1.0.0-src.tar.gz.sha512

Mac:
shasum -a 512 apache-doris-shade-1.0.0-src.tar.gz > apache-doris-shade-1.0.0-src.tar.gz.sha512
```

最终得到三个文件：

```
apache-doris-shade-1.0.0-src.tar.gz
apache-doris-shade-1.0.0-src.tar.gz.asc
apache-doris-shade-1.0.0-src.tar.gz.sha512
```

将这三个文件移动到 svn 目录下：

```
doris/doris-shade/1.0.0/
```

最终 svn 目录结构类似：

```
├── 1.2.3-rc01
│   ├── apache-doris-1.2.3-src.tar.gz
│   ├── apache-doris-1.2.3-src.tar.gz.asc
│   ├── apache-doris-1.2.3-src.tar.gz.sha512
...
├── KEYS
├── doris-shade
│   └── 1.0.0
│       ├── apache-doris-shade-1.0.0-src.tar.gz
│       ├── apache-doris-shade-1.0.0-src.tar.gz.asc
│       └── apache-doris-shade-1.0.0-src.tar.gz.sha512
```

其中 1.2.3-rc01 是 Doris 主代码的目录，而 `doris-shade/1.0.0` 下就是本次发布的内容了。

注意，KEYS 文件的准备，可参阅 [发版准备](./release-prepare.md) 中的介绍。

### 4. 投票

在 dev@doris 邮件组发起投票，模板如下：

```
Hi all,

This is a call for the vote to release Apache Doris-Shade 1.0.0

The git tag for the release:
https://github.com/apache/doris-shade/releases/tag/doris-shade-1.0.0

Release Notes are here:
https://github.com/apache/doris-shade/blob/doris-shade-1.0.0/CHANGE-LOG.txt

Thanks to everyone who has contributed to this release.

The release candidates:
https://dist.apache.org/repos/dist/dev/doris/doris-shade/

KEYS file is available here:
https://downloads.apache.org/doris/KEYS

To verify and build, you can refer to following link:
https://doris.apache.org/community/release-and-verify/release-verify

The vote will be open for at least 72 hours.

[ ] +1 Approve the release
[ ] +0 No opinion
[ ] -1 Do not release this package because …
```

## 完成发布

请参阅 [完成发布](./release-complete.md) 文档完成所有发布流程。

## 附录：发布到 SNAPSHOT

Snapshot 并非 Apache Release 版本，仅用于发版前的预览。在经过 PMC 讨论通过后，可以发布 Snapshot 版本

切换到 doris shade 目录

```
mvn deploy
```

之后你可以在这里看到 snapshot 版本：

```
https://repository.apache.org/content/repositories/snapshots/org/apache/doris/doris-shade/
```
