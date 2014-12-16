# git-cc

Simple bridge between base Clearcase or UCM and Git.
简单的桥梁在CC 或者UCM和Git之间

## Warning，警告

我写这个纯粹是为了好玩，看看能不能一次解决停用Clearcase的问题
I wrote this purely for fun and to see if I could stop use Clearcase at work
once and for all.

我将会继续的改进这工具来满足我自己的需求，但是我更希望看到它能被应用到别人的工作中。
I will probably continue to hack away at it to suite my needs, but I would
love to see it get some real-world polish. (Actually what I would love to see
more is for Clearcase to die, but don't think that's going to happen any time
soon).

欢迎建议和来稿。
Suggestions on anything I've done are more than welcome.

我最近做了点改动来支持添加那些用了git-cat的二进制文件的。不幸的是，git-cat并不支持处理行尾标示符，所以我的gitcc初始化设置core.autocrlf为false。这个只和windows系统用户有关。在你第一次提交之后，也不要改动这个设定，不然会带来更多麻烦。为此困扰的用户我深表歉意。
Also, I have made a change recently to support adding binary files which uses
git-cat. Unfortunately git-cat doesn't handle end line conversions and so I
have made gitcc init set core.autocrlf to false. This is only relevant for
Windows users. Don't try changing this after your first commit either as it
will only make matters worse. My apologies to anyone that is stung by this.

## Workflow，工作过程

Initialise:

    git init
    gitcc init d:/view/xyz
    gitcc rebase
    # Get coffee
    # Do some work
    git add .
    git commit -m "I don't actually drink coffee"
    gitcc rebase
    gitcc checkin

Initialise (fast):较快

Rebase can be quite slow initially, and if you just want to get a snapshot of
Clearcase, without the history, then this is for you:
第一次rebase会非常慢，如果你仅仅是需要CC的snapshot而不需要历史的话，如下这样：

    gitcc init d:/view/xyz
    gitcc update "Initial commit"

Other:
其他

These are two useful flags for rebase which is use quite frequently.
rebase命令中很有用的两个标签

    gitcc rebase --stash

Runs stash before the rebase, and pops it back on afterwards.
在rebase之前做一下stash, 先保存一下现场，之后要用的时候再重现。。。。。。这里是什么个意思，没明白？？？？？？？？？？？

    gitcc rebase --dry-run

Prints out the list of commits and modified files that are pending in clearcase.
打印出clearcase中，已经commit和修改的那些文件

To synchronise just a portion of your git history (instead of from the
very first commit to HEAD), mark the start point with the command:
来同步一部分git历史，不是从第一次commit到现在，而是从某某tag开始。

    gitcc tag <commit>

To specify an existing Clearcase label while checking in, in order to let your
dynamic view show the version of the element(s) just checked in if your
confspec is configured accordingly, use the command:
不太了解Clearcase，实在是没有搞明白？？？？？？？？？？？？？？？？？？？？？？
关键词，label， dynamic，check in, conspec

    gitcc checkin --cclabel=YOUR_EXISTING_CC_LABEL

Note that the CC label will be moved to the new version of the element, if it is already used.
还是没搞明白？？？？？？？？？？？？？？？？？？？？？？？？？？？？？

## Configuration，配置方法

You need to add a mapping for each user in your clearcase history to users.py.
你需要把claercase历史中的user都写到user.py文件中
You can also limit which branches and folders you import from.
你可以限制从哪个分支和文件夹导入
eg. .git/gitcc

    [core]
    include = FolderA|FolderB
    exclude = FolderA/sub/folder|FolderB/other/file
    debug = False
    type = UCM
    [master]
    clearcase = D:\views\co4222_flex\rd_poc
    branches = main|ji_dev|ji_*_dev|iteration_*_dev
    [sup]
    clearcase = D:\views\co4222_sup\rd_poc
    branches = main|sup

In this case there are two separate git branches, master and sup, which
correspond to different folders/branches in clearcase.

## Notes，注意事项

Can either work with static or dynamic views. I use dynamic at work because
it's much faster not having to update. I've done an update in rebase anyway,
just-in-case someone wants to use it that way.
静态动态view都可以，用动态view更新会快一些，因为不需要更新。

Can also work with UCM, which requires the 'type' config to be set to 'UCM'.
This is still a work in progress as I only recently switched to this at work.
Note the the history is still retrieved via lshistory and not specifically from
any activity information. This is largely for convenience for me so I don't have
to rewrite everything. Therefore things like 'recommended' baselines are ignored.
I don't know if this will cause any major dramas or not.
也可以和UCM一起用，设定好type就是了。不过这仍然是我最近正在进行改进的地方。注意到这些历史仍然是通过lshistory拿到，而不是其他activity信息。所以一些‘推荐的’baselines就被忽略了。我不知道这会不会引起其他的问题。

## Troubleshooting

1. WindowsError: [Error 2] The system cannot find the file specified

You're most likely running gitcc under Windows Cmd. At moment this isn't
supported. Instead use Git Bash, which is a better console anyway. :-)

If you have both msysgit and Cygwin installed then it may also be
[this](https://github.com/charleso/git-cc/issues/10) problem.

2. cleartool: Error: Not an object in a vob: ".".

The Clearcase directory you've specified in init isn't correct. Please note
that the directory must be inside a VOB, which might be one of the folders
inside the view you've specified.

3. fatal: ambiguous argument 'clearcase': unknown revision or path not in the working tree.

If this is your first rebase then please ignore this. This is expected.

4. pathspec 'master_cc' did not match any file(s) known to git

See Issue [8](https://github.com/charleso/git-cc/issues/8).

## Behind the scenes，附言

A smart person would have looked at other git bridge implementations for
inspiration, such as git-svn and the like. I, on the other hand, decided to go
cowboy and re-invent the wheel. I have no idea how those other scripts do their
business and so I hope this isn't a completely stupid way of going about it.
我就是要自己重新发明轮子，我不知道它们的脚本如何工作的，不过希望我的这个工具不会是比较笨拙的方法。

I wanted to have it so that any point in history you could rebase on-top of the
current working directory. I've done this by using the clearcase commit time
for git as well. In addition the last rebased commit is tagged and is used
to limit the history query for any chances since. This tagged changeset is
therefore also used to select which commits need to be checked into clearcase.
我希望在git中可以找到clearcase的每一个历史节点。我用clearcase的commit时间来作为git的commit时间。
另外，tagged是什么意思？？？？？？？？？？？？？？

## Problems

It is worth nothing that when initially importing the history from Clearcase
that files not currently in your view (ie deleted) cannot be reached without
a config spec change. This is quite sad and means that the imported history is
not a true one and so rolling back to older revisions will be somewhat limited
as it is likely everything won't compile. Other Clearcase importers seem
restricted by the same problem, but none-the-less it is most frustrating. Grr!
在导入历史信息时有些无用功：有些文件在现在的clearcase的view中已经删除了，这样的文件只有通过修改configspec文件来获得。这样导入的历史就不符合了（估计是导入之后在git的历史中这些文件是不存在的），一些老版本中可能所有文件都无法编译了。其他的clearcase导入工具也会因此而有所限制，尽管如此，这才是最烦人的
