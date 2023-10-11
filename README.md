# 多个仓库 如何 共享某一模块/代码
How do multiple repos share dir/code/module, some solutions as below:
* git subtree
* git fetch && checkout方式 merge
* soft link (电脑上建立目录软链)
* git submodule
* multiple workspace

## dir structure && description

```
├── README.md                 当前文件
└── app                       应用代码目录
    ├── base_shared           基础设施代码 **与其它项目可共享的**
    │   ├── components
    │   │   └── list
    │   │       └── base.js
    │   └── utils
    │       └── theme
    │           └── index.js
    ├── common                 常用组件、工具代码  非共享的、MainRepoDemo才会用到的
    │   ├── components
    │   │   └── navigationBar
    │   │       └── index.js
    │   └── utils
    │       └── time
    │           └── index.js
    └── modules                模块集合，存放MainRepoDemo用到的各业务模块 
        ├── graphic_shared     graphic模块 **共享自(来自于)其他项目**
        └── home
            └── index.js
```


## 方式1： git subtree 合并某仓库 ~~的指定目录~~ 到 另一仓库的指定目录

### 尝试 指定子仓库的指定目录
* 添加 [GraphicRepoDemo](https://github.com/zyestin/GraphicRepoDemo) 仓库作为远程仓库，以便后续使用它的内容
```
git remote add graphicproject-remote git@github.com:zyestin/GraphicRepoDemo.git
```

sourceTree效果图
<img width="914" alt="image" src="https://github.com/zyestin/MainRepoDemo/assets/51897571/464c765f-bdbf-44b0-ac14-e4eab1abfc91">

* 使用 Git subtree 将 "[GraphicRepoDemo](https://github.com/zyestin/GraphicRepoDemo)" 的 "app/modules/graphic_shared" 目录合并到 "MainRepoDemo" 仓库的相同路径下。运行以下命令：
```
git subtree add --prefix=app/modules/graphic_shared graphicproject-remote master --squash
```
但结果却是 把`GraphicRepoDemo`全部代码 合并到了 `MainRepoDemo`的"app/modules/graphic_shared" 目录下。 确实做到了commit信息继续，而不产生新的commit信息
<img width="1202" alt="image" src="https://github.com/zyestin/MainRepoDemo/assets/51897571/e873343a-7f56-4eb3-84d7-f239e4980c12">

既然希望commit信息继续保留（**范围是整个GraphicRepoDemo仓库**） 而不会产生新的commit，又希望指定仓库中的一个子目录"app/modules/graphic_shared"（**范围并不是整个仓库**）来merge，看来本该就是做不到了，不管用不用`subtree`

于是，需要将`graphic_shared`单独作为一个仓库，供`MainRepoDemo`和`GraphicRepoDemo`用subtree方式享用，即可~

### 尝试 将子目录抽出来作为单独子仓库 去作为整体共享

* 将`GraphicRepoDemo`下"app/modules/graphic_shared"内的文件转移到 [GraphicRepo](https://github.com/zyestin/GraphicRepo)

* 删掉`MainRepoDemo`和`GraphicRepoDemo`下"app/modules/graphic_shared"

拿[MainRepoDemo](https://github.com/zyestin/MainRepoDemo)试一试
* 移除`MainRepoDemo`之前的subtree 
```
git remote remove graphicproject-remote
```

* 为`MainRepoDemo`添加子仓库
```
git remote add graphicrepo git@github.com:zyestin/GraphicRepo.git
git subtree add --prefix=app/modules/graphic_shared graphicrepo main --squash
```
这样，

* 在父仓库`MainRepoDemo`中更新了子仓库中的内容，同步到子仓库
```
//先提交父仓库
git add
git commit

//再提交到子仓库
git subtree push --prefix=app/modules/graphic_shared graphicrepo main          
git push using:  graphicrepo main
Enumerating objects: 55, done.
Counting objects: 100% (55/55), done.
Delta compression using up to 8 threads
Compressing objects: 100% (42/42), done.
Writing objects: 100% (50/50), 7.40 KiB | 2.47 MiB/s, done.
Total 50 (delta 10), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (10/10), completed with 1 local object.
To github.com:zyestin/GraphicRepo.git
   0cb8da2..ff099bb  ff099bbbc7ac3939ec0073f1bdc302fa4f5bfc26 -> main
```
可以看到子仓库，有许多待拉取的，吓人
<img width="1269" alt="image" src="https://github.com/zyestin/MainRepoDemo/assets/51897571/6b73e49d-531b-45af-bd0e-c703bd70da90">

* 子仓库 pull
可以看到，在父仓库对子仓库做的修改，已经同步过来了
<img width="1087" alt="image" src="https://github.com/zyestin/MainRepoDemo/assets/51897571/c9ef0ec3-7823-4bf6-94b3-aa97fa72ccb8">

可见，虽然在父仓库对子仓库做的修改后 可以通过`subtree push`将更改同步到子仓库，但是 **子仓库的commit信息，竟然被污染了！！！**


## 方式2：`git fetch && checkout`方式merge
执行如下命令
```
git fetch graphicproject-remote
git checkout graphicproject-remote/main -- app/modules/graphic_shared
```
类似于手动 从`GraphicRepoDemo`把"app/modules/graphic_shared"拷贝，黏贴到`MainRepoDemo`的该目录


## 方式3：软链
在本地`MainRepoDemo`的"app/modules/graphic_shared"，通过软链指向`GraphicRepoDemo`的"app/modules/graphic_shared"

注意：`.gitignore`中需要忽略"app/modules/graphic_shared"，防止提交了这个软链空壳文件上去

## 方式4：git submodule 
拿[GraphicRepoDemo](https://github.com/zyestin/MainRepoDemo)试一试 往里面关联 一个新的子仓库[Graphic2Repo](https://github.com/zyestin/Graphic2Repo)

### 往`GraphicRepoDemo`的"app/modules/graphic_shared"目录，拉取子仓库`Graphic2Repo`代码
```
pwd                                 
/Users/yestin/Desktop/gitfiles/RepoShare/GraphicRepoDemo

git submodule add git@github.com:zyestin/Graphic2Repo.git app/modules/graphic_shared
```
<img width="1146" alt="image" src="https://github.com/zyestin/MainRepoDemo/assets/51897571/3e16d68f-7b7f-41f4-9023-ee34578df4a5">
`git submodule add`这个关联操作，只会新增两个改动`.gitmodules`、`app/modules/graphic_shared` （而不会将`graphic_shared`内部的各个文件都作为新增改动）

实际的`app/modules/graphic_shared`目录下的文件，是与 子仓库`Graphic2Repo`代码 一致的
<img width="856" alt="image" src="https://github.com/zyestin/MainRepoDemo/assets/51897571/d8fbbc33-4375-4002-b711-a42c96a70e68">


### 子仓库`Graphic2Repo`代码更新，更新 父仓库`GraphicRepoDemo`的"app/modules/graphic_shared"目录代码
```
git submodule update --remote app/modules/graphic_shared
Submodule path 'app/modules/graphic_shared': checked out '68e524d1b15e3d678a094e83e6a02d3cb8116fc8'
```
`git status`变化：仅`app/modules/graphic_shared`的`commit id`变化了
<img width="509" alt="image" src="https://github.com/zyestin/MainRepoDemo/assets/51897571/8ea4787a-2113-4942-8a1b-8bcbd6eb7307">

实际代码变化：父仓库`app/modules/graphic_shared`内代码与子仓库一致了
<img width="884" alt="image" src="https://github.com/zyestin/MainRepoDemo/assets/51897571/b16cec7d-1507-46c4-ae1f-f7b8002a17e8">

### 在父仓库更改子仓库代码，将改动更新到子仓库远端
* 更改代码，保存
    * 更改前
    <img width="400" alt="image" src="https://github.com/zyestin/MainRepoDemo/assets/51897571/88d7e35c-3002-4d1d-bebc-52f9743f1d86">
    * 更改后
    <img width="400" alt="image" src="https://github.com/zyestin/MainRepoDemo/assets/51897571/2dfb4f85-e367-466d-b904-a9b9cb0bda79">
发现，此时仅子仓库的`git status`有变化。
> 对比subtree 就会发现在主仓库也发现代码变更信息，那么用subtree就明显不合适了
>  <img width="452" alt="image" src="https://github.com/zyestin/MainRepoDemo/assets/51897571/a1ab3d69-fae2-4662-a4aa-1d5a94591f72">

* commit
```
cd app/modules/graphic_shared
git add .
git commit -m "test: 在父仓库更改子仓库代码，更新远端子仓库代码"
```
commit后，父仓库下`git status`就有变化了，`app/modules/graphic_shared`的`commit id`变化了
<img width="484" alt="image" src="https://github.com/zyestin/MainRepoDemo/assets/51897571/80a80098-91a4-4f2f-9b9c-31258d2248f8">

* push  子仓库的改动同步到远端子仓库
```
///push到父仓库
...

///push子仓库
cd app/modules/graphic_shared
//... add commit ... 可直接在VSC操作
git push origin HEAD:graphic2-remote       //可以指定 main 或别的 远程分支
Enumerating objects: 10, done.
Counting objects: 100% (10/10), done.
Delta compression using up to 8 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (6/6), 629 bytes | 629.00 KiB/s, done.
Total 6 (delta 2), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (2/2), completed with 1 local object.
To github.com:zyestin/Graphic2Repo.git
   16b612c..0833e03  HEAD -> graphic2-remote
```
下图可见，子仓库的git log非常纯粹，commit 都是子仓库代码改动信息
<img width="1014" alt="image" src="https://github.com/zyestin/MainRepoDemo/assets/51897571/06835f74-5d71-4863-8fab-e1e51e8c0adc">

> 一般`git push`前还是先`git pull`一次，以免他人push了，就会报如下错误  
> 报这个错的话，就执行`git config pull.rebase false`，然后再`pull`
```
git pull origin graphic2-remote
From github.com:zyestin/Graphic2Repo
 * branch            graphic2-remote -> FETCH_HEAD
hint: You have divergent branches and need to specify how to reconcile them.
hint: You can do so by running one of the following commands sometime before
hint: your next pull:
hint: 
hint:   git config pull.rebase false  # merge
hint:   git config pull.rebase true   # rebase
hint:   git config pull.ff only       # fast-forward only
hint: 
hint: You can replace "git config" with "git config --global" to set a default
hint: preference for all repositories. You can also pass --rebase, --no-rebase,
hint: or --ff-only on the command line to override the configured default per
hint: invocation.
```



## 参考
* [Subtree 与 Submodule](https://gb.yekai.net/concepts/subtree-vs-submodule)
> commit: 父仓库直接提交父子仓库目录里的变动。若修改了子仓库的文件，则需要执行 subtree push

* [Git subtree用法与常见问题分析](https://zhuanlan.zhihu.com/p/253148857)
> 四个方案个人认为的排序是subtree = submodule > dll > npm


