# 为什么我的master分支不在了？


### 为什么写这篇文章

这一次有一个新的需求要开发，然而我们项目还在测试中，项目代码都在dev上面，所以在本地
新建一个分支feature。并切换到feature分支上，在feature愉快的开发。所有新开发的代码
都没有add。准备项目上线后，把feature分支的代码直接merge到dev上面。突然，测试发现了一个
问题，我需要在切换到dev分支上面修改bug。既然有bug需要修改，那就切换到dev修改bug。

按我的想法，当我切换到dev上面，我的feature分支应该都看不见。dev上的代码应该是我没开发新
需求的代码。可是当我切换到dev的时候。发现我在feature分支上面所有的更改都被看见了。此时，
只想问为什么呢？所以才有了这边篇文章。


### 我的第一步操作--master上面的操作

在我们工作目录里面，新建一个test目录，然后使用git init 初始化了项目
在git中，默认的初始化的分支是master，在master我们执行如下步骤：

```
1. touch 1.txt---新建一个空的txt文件
2. 在空文件里面写入如下内容：我是在master上面的写的内容的
```

### 我的第二步--切换到dev分支上面操作

然后，我创建了分支dev,并且切换到dev分支，做了以下操作：

```
1. touch 2.txt---新建一个空的txt文件
2. 在空文件写入如下内容： 我是在dev上面的写的内容，这个内容没有add commit，只是添加 修改了内容
```

### 我的第三步---切换到master分支


```
1. git chekout master
2. 报错了---告诉我切换不了。
3. 错误信息---error: pathspec 'master' did not match any file(s) known to git.
```


### 此时，我就上网查了一下为什么

我看到了这句话：
还没有提交过，即没有commit过任何文件，当commit过以后就可以切换分支了。
既然这样，那我就在dev上面执行了下面的操作

```
1. git add .
2. git commit -m "提交信息"
```

我想，这个时候我可以切换了吧。开心的执行git checkout master
结果还是一样的错误。到底是什么原因呢。看来还得重新找一下真正的原因。
结果搜索到这篇文章。
[git checkout master error: pathspec 'master' did not match any file(s) known to git](https://stackoverflow.com/questions/30459631/git-checkout-master-error-pathspec-master-did-not-match-any-files-known-to)


### 总结

我们的第一个分支往往是master，而在master上面我们没有做什么提交commit，
然后我们切换到dev上面，在dev上面做了commit-----切换到master出错。

这个原因是因为，我们的第一个分支即master，需要在首次commit之后，才会被创建。
问我们没有做什么commit，就切换分支了到dev。所以master分支是不存在的，没有被创建的。
如何证明。此时我能使用查看分支的命令：
```
	git branch
```
我们看见只有dev分支， 而没有master分支，这就印证这个结论。

可是还有一个问题，为什么两个分支的内容同步了？这回在下一遍文章回答。

