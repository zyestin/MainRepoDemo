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

### 添加 [GraphicRepoDemo](https://github.com/zyestin/GraphicRepoDemo) 仓库作为远程仓库，以便后续使用它的内容
```
git remote add graphicproject-remote git@github.com:zyestin/GraphicRepoDemo.git
```

sourceTree效果图
<img width="914" alt="image" src="https://github.com/zyestin/MainRepoDemo/assets/51897571/464c765f-bdbf-44b0-ac14-e4eab1abfc91">

### 使用 Git subtree 将 "[GraphicRepoDemo](https://github.com/zyestin/GraphicRepoDemo)" 的 "app/modules/graphic_shared" 目录合并到 "MainRepoDemo" 仓库的相同路径下。运行以下命令：
```
git subtree add --prefix=app/modules/graphic_shared graphicproject-remote master --squash
```
但结果却是 把`GraphicRepoDemo`全部代码 合并到了 `MainRepoDemo`的"app/modules/graphic_shared" 目录下。 确实做到了commit信息继续，而不产生新的commit信息
<img width="1202" alt="image" src="https://github.com/zyestin/MainRepoDemo/assets/51897571/e873343a-7f56-4eb3-84d7-f239e4980c12">

既然希望commit信息继续保留（**范围是整个GraphicRepoDemo仓库**） 而不会产生新的commit，又希望指定仓库中的一个子目录"app/modules/graphic_shared"（**范围并不是整个仓库**）来merge，看来本该就是做不到了，不管用不用`subtree`

于是，需要将`graphic_shared`单独作为一个仓库，供`MainRepoDemo`和`GraphicRepoDemo`用subtree方式享用，即可~


## 方式2：`git fetch && checkout`方式merge
执行如下命令
```
git fetch graphicproject-remote
git checkout graphicproject-remote/main -- app/modules/graphic_shared
```
类似于手动 从`GraphicRepoDemo`把"app/modules/graphic_shared"拷贝，黏贴到`MainRepoDemo`的该目录


## 方式3：软链
在本地`MainRepoDemo`的"app/modules/graphic_shared"，通过软链指向`GraphicRepoDemo`的"app/modules/graphic_shared"

