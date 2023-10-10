# MainRepoDemo's README.md

## dir structure && description


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






