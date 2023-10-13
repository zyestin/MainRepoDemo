# 多个仓库 如何 共享某一模块/代码/仓库
How do multiple repos share dir/code/module/repo, some solutions as below:
* git subtree  
  较强不可忍受点：在VSC主仓库修改了子仓库代码，subtree push同步子仓库改动到远端后，发现子仓库的commit记录会含有主仓库的commit记录，被严重污染。。。
* git fetch && checkout方式 merge 子仓库代码 到 主仓库指定的目录  
  很强不可忍受点：与手动从子仓库拷贝代码到主仓库方式一样，这些diff，又会出现在主仓库 又需要重新在主仓库commit。。。
* soft link (电脑上建立目录软链)  
  较强不可忍受点：.gitignore是忽略了主仓库中的 子仓库的所有代码的，相当于远端代码库 缺了这一块，然而，连这块的关联信息 该如何解决？ 
  一般不可忍受点：在VSC主仓库修改了子仓库代码，（可以忍受：需要去真正的子仓库去提交代码），就必然会多开一个子仓库窗口 进行git操作。
* git submodule  
  需要注意地方：在主仓库中修改子仓库，通过命令先提交子仓库代码，再提交主仓库代码。认为这是顺理成章的，因为子仓库代码改动，本该独立更新
  > 越来越觉得，这是软链的增强版
  > 1. 主仓库远端，缺失的子仓库这一块，但`.gitmodule`能表示与具体子仓库的关联关系
  > 2. 在VSC主仓库修改了子仓库代码，可以在主仓库，打包几个命令，做到在主仓库直接维护子仓库。
* multiple workspace  
  暂未实践

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

* push 子仓库的改动同步到远端子仓库
  * 通过命令
    ```
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
  * 通过VSC也可以push子仓库代码
    <img width="1340" alt="image" src="https://github.com/zyestin/MainRepoDemo/assets/51897571/81d10634-660e-4953-8411-ea63f5310524">
    > VSC push后，确实子仓库远端ok了
    > <img width="1276" alt="image" src="https://github.com/zyestin/MainRepoDemo/assets/51897571/7e0442c0-d856-4b2a-94ea-3b2e6d920ae9">

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

### 多人协作 在主仓拉取子仓代码

* 对于首次，直接操作`submodule update` ?  不，首次需要先操作`git submodule update --init`  
  他人首次拉取[GraphicRepoDemo](https://github.com/zyestin/GraphicRepoDemo)后，会发现`app/modules/graphic_shared`目录下的代码是空的，看来需要额外的操作去同步这块代码
```
git submodule update --init --recursive                15:30:27
Submodule 'app/modules/graphic_shared' (git@github.com:zyestin/Graphic2Repo.git) registered for path 'app/modules/graphic_shared'
Cloning into '/Users/yestin/Desktop/gitfiles/RepoShare/GraphicRepoDemo2/app/modules/graphic_shared'...
Submodule path 'app/modules/graphic_shared': checked out '6ee3837c117a0f9b2696f3f0b3099ff21333b235'
```
操作完`git submodule update --init --recursive` 发现`app/modules/graphic_shared`目录下的代码已经同步过来了
<img width="1161" alt="image" src="https://github.com/zyestin/MainRepoDemo/assets/51897571/46239afe-e019-4ec3-9ffb-c742beb7648b">

> 直接操作，会出现什么问题呢？会有如下提示
```
git submodule update --remote app/modules/graphic_shared
Submodule path 'app/modules/graphic_shared' not initialized
Maybe you want to use 'update --init'?
```

* 非首次，操作`git submodule update --remote app/modules/graphic_shared`
    * 当一个同事更新了 子仓库[Graphic2Repo](https://github.com/zyestin/Graphic2Repo) 其他同事如何感知呢？  
    使用sourcetree的同事，在主仓库是感知不到有未pull的commit的，（VSC本来就不支持提示有N个待pull的commit）
    此时可以打开VSC的`commit graph`可视化窗口，能看出是否有待pull的commit，如下图
    <img width="1230" alt="image" src="https://github.com/zyestin/MainRepoDemo/assets/51897571/886ea7f5-8844-4474-9ebc-b53825ce7175">
    * 执行 在主仓库拉取子仓库代码
```
git submodule update --remote app/modules/graphic_shared
Submodule path 'app/modules/graphic_shared': checked out '3ab3f32779995d68522878d563dfb63bfbd5f2ab'
```
> 本以为 .gitmodule文件中记录了 `path = app/modules/graphic_shared`，以为直接执行`git submodule update`，不用指定路径了，
> 但依然需要执行完整的`git submodule update --remote app/modules/graphic_shared`

### 多人协作 子仓库多人次修改和提交

* 没有冲突时  
在主仓改子仓，commit但不push；  
在子仓改，commit且push
<img width="1521" alt="image" src="https://github.com/zyestin/MainRepoDemo/assets/51897571/d86ebc33-a80d-4040-9464-762ffd17ea1a">
然后，在主仓先pull后push，即点击`Sync Changes`
<img width="1127" alt="image" src="https://github.com/zyestin/MainRepoDemo/assets/51897571/d0731390-59db-4f74-b4df-ea8758a768c4">
会报错一次，如下，执行一次`git config pull.rebase false`就好了，以后`Sync Changes`将不再被中断，直接会进行merge
```
hint: You have divergent branches and need to specify how to reconcile them.
...
```
VSC 通过`Sync Changes`执行 pull && push后，本地、远程子仓库更新成功
<img width="1363" alt="image" src="https://github.com/zyestin/MainRepoDemo/assets/51897571/d3f9f9d2-b976-4b15-93bd-6a52a25719f3">

* 有冲突时
VSC操作`Sync Changes`后，正常走熟悉的merge流程，标记出了冲突代码
<img width="1204" alt="image" src="https://github.com/zyestin/MainRepoDemo/assets/51897571/829fb90c-ba72-475d-862b-627b904016f5">
然后，修复掉冲突代码，操作 `stage`、`commit`、`Sync Changes`就可以了


## 参考
* [Subtree 与 Submodule](https://gb.yekai.net/concepts/subtree-vs-submodule)
> commit: 父仓库直接提交父子仓库目录里的变动。若修改了子仓库的文件，则需要执行 subtree push

* [Git subtree用法与常见问题分析](https://zhuanlan.zhihu.com/p/253148857)
> 四个方案个人认为的排序是subtree = submodule > dll > npm

* GPT 对比 subtree与submodule
> ![image](https://github.com/zyestin/MainRepoDemo/assets/51897571/50b20661-707c-46b7-93b7-a76547e13878)

