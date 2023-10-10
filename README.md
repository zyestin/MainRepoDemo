# How do multiple repos share dir/code/module

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


## 参考
[Subtree 与 Submodule](https://gb.yekai.net/concepts/subtree-vs-submodule)
