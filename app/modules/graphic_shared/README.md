# GraphicRepoDemo
graphic project demo，whose graphic_shared module should be shared to [MainRepoDemo](https://github.com/zyestin/MainRepoDemo)




## dir structure && description

```
.
├── README.md
└── app
    ├── common
    │   ├── components
    │   │   └── navigationBar
    │   │       └── index.js
    │   └── utils
    │       └── time
    │           └── index.js
    └── modules
        ├── graphic_shared                            该模块需要共享给 MainRepoDemo
        │   ├── assets
        │   │   ├── animations
        │   │   │   └── loading.json
        │   │   └── images
        │   │       ├── create
        │   │       │   └── collection_selected.png
        │   │       └── feedDetail
        │   │           └── icon-like.png
        │   ├── components
        │   │   ├── comment
        │   │   │   └── index.js
        │   │   └── feed
        │   │       └── index.js
        │   ├── pages
        │   │   ├── follow
        │   │   │   └── index.js
        │   │   ├── home
        │   │   │   └── index.js
        │   │   └── mine
        │   │       └── index.js
        │   └── utils
        │       └── time
        │           └── index.js
        └── login

```
