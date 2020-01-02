
工作流： git add 将本地修改加入暂存区 -> commit -> 风格、规范检查 -> 生成版本 -> 日志生成 -> push

## 在提交前对提交部分快照进行风格检查 

husky + lint-staged + eslint + prettier

### husky  

使用 [husky](https://github.com/typicode/husky/blob/master/) 来管理提交的修改，可以在 git commit 之前再校验一次代码，达到使用 `pre-commit + shell/ruby`等脚本进行检查的效果, exit 不为0 则中断提交

```
  "husky": {
    "hooks": {
      "pre-commit": "prettier -c"
    }
  },
```
如此配置会将提项目中的所有文件进行一次 prettier 检查，浪费了时间在无关文件上。解决方案，往下瞅

### lint-staged

[lint-staged github](https://github.com/okonet/lint-staged) 

其中 `staged` 是 `Git` 里面的概念，指待提交区(暂存区)， `git add` 将代码修改移入暂存区。 

只会校验提交里面指定需要校验的文件

```json
    "husky": {
      "hooks": {
        "pre-commit": "lint-staged"
    }
  },
  // 指定文件及校验规则
  "lint-staged": {
    "src/*.js": [
      "eslint --ext .js",
      "eslint --fix",
      "prettier --check 'src/*.js'"
    ],
    "*.css": [
      "prettier --check"
    ]
  },
```

## 版本控制 

版本号： MAJOR.MINOR.PATCH

[standard-version](https://github.com/conventional-changelog/standard-version) / [release-it](https://github.com/release-it/release-it)

`standard-version` 是一款遵循语义化版本[(semver)](https://semver.org/)和 `commit message` 标准规范 的版本和 `changlog` 自动化工具。 
通常情况下，我们会在 master 分支进行如下的版本发布操作：
1. git pull origin master
2. 根据 pacakage.json 中的 version 更新版本号，更新 changelog
3. git add -A, 然后 git commit
4. git tag 打版本操作
5. push 版本 tag 和 master 分支到仓库

其中2，3，4则是 standard-version 工具会自动完成的工作，配合本地的 shell 脚本，则可以自动完成一系列版本发布的工作了。

## CHANGELOG 生成 

CHANGELOG只有在发版时刻才生成，前提是规范化提交  
在 CHANGELOG.md 的头部加上自从上次发布版本以来的变动。

- commitizen  
    提供的 git cz 命令替代 git commit 命令, 帮助生成符合规范的 commit message  

- cz-conventional-changelog  
    适配器、不同的项目本身的构建方式的不同，commitizen 支持不同适配器的扩展，从而去满足不同的构建需求的。一个符合 Angular 团队规范的 preset  

- conventional-changelog-cli  
    默认推荐的 commit 标准是来自 `angular`  
    目前集成了包括 `atom`, `codemirror`, `ember`, `eslint`, `express`, `jquery` 等项目的标准

- conventional-changelog-custom-config  
    [github 地址](conventional-changelog-custom-config)  
    实现自定义的 CHANGELOG 需要该插件支持自定义

    **script命令**：  
    `conventional-changelog -p custom-config -i CHANGELOG.md -s -r 0 -n ./changelog-option.js `  
    参数如下：  

    | 选项 |      详情       |                                      说明                                      |
    | :--- | :-------------: | :----------------------------------------------------------------------------: |
    | -p   |    --preset     |         使用的模版、默认是 angular，-p custom-config 使用自定义的配置          |
    | -i   |    --infile     |                       Read the CHANGELOG from this file                        |
    | -s   |   --same-file   | Outputting to the infile so you don't need to specify the same file as outfile |
    | -r   | --release-count |   从最新生成的版本数如果为0，将重新生成整个变更日志，并覆盖输出文件默认值：1   |
    | -n   |    --config     |                               指定配置文件的路径                               |

    更多请使用👉 `--help`

- standard-version 
    包含版本管理和生成日志的功能

```json
"scripts": {
    "commit": "prettier --check 'src/*.js' && git cz",
    "lint": "eslint --ext .js",
    "clog": "conventional-changelog -p custom-config -i CHANGELOG.md -s -r 0 -n ./changelog-option.js",
    "prettier": "prettier --check --write 'src/*.js' 'static/*.css'",
    "release:major": "standard-version -r major",
    "release:minor": "standard-version -r minor",
    "release:patch": "standard-version -r patch",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "commitizen": {
    "path": "cz-conventional-changelog"
  },
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "src/*.js": [
      "eslint --ext .js",
      "eslint --fix",
      "prettier --check 'src/*.js'"
    ],
    "*.css": [
      "prettier --check"
    ]
  },
  "devDependencies": {
    "commitizen": "^4.0.3",
    "conventional-changelog-cli": "^2.0.31",
    "conventional-changelog-custom-config": "^0.2.0",
    "cz-conventional-changelog": "^3.0.2",
    "eslint": "^6.8.0",
    "husky": "^3.1.0",
    "lint-staged": "^9.5.0",
    "prettier": "^1.19.1",
    "standard-version": "^7.0.1"
  }
```




--- 

## commit 类型相关 

feat：新功能  
fix：修补 bug  
docs：修改文档，比如 README, CHANGELOG, CONTRIBUTE 等等  
style：不改变代码逻辑 (仅仅修改了空格、格式缩进、逗号等等) 并不是修改css style样式  
refactor：重构（既不修复错误也不添加功能）  
perf: 优化相关，比如提升性能、体验  
test：增加测试，包括单元测试、集成测试等  
build: 构建系统或外部依赖项的更改  
ci：自动化流程配置或脚本修改  
chore: 非 src 和 test 的修改（其他修改）  
revert: 恢复先前的提交  



## Git Hooks相关  

git hook 存在于项目中 .git 文件夹下的 hooks中，默认以点 sample 扩展名存在，不会被 git 管理到，使用hook进行控制git流程需要另外在项目中保存hooks  

| 钩子               |                      阶段                      |                             用途                              |            |
| :----------------- | :--------------------------------------------: | :-----------------------------------------------------------: | :--------: |
| pre-commit         |                 键入提交信息前                 | 对提交的快照进行检查 可以用 `git commit --no-verify`绕过该环节 | 客户端钩子 |
| prepare-commit-msg | 启动提交信息编辑器之前，默认信息被创建之后运行 |                                                               | 客户端钩子 |
| commit-msg         |  接收一个存有当前提交信息的临时文件的路径参数  |            用来在提交通过前验证项目状态或提交信息             | 客户端钩子 |
| post-commit        |             整个提交过程完成后运行             |                       一般用于通知之类                        | 客户端钩子 |
| post-merge         |             git merge 成功运行之后             |                                                               | 客户端钩子 |


## TODO： 服务端钩子

| 钩子        |                            阶段                            |                                                                                    用途                                                                                     |            |
| :---------- | :--------------------------------------------------------: | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | :--------: |
| pre-recive  | 处理来自客户端的推送操作时，最先被调用的脚本是 pre-receive | 可以用这个钩子阻止对引用进行非快进（non-fast-forward）的更新，或者对该推送所修改的所有引用和文件进行访问控制                                           对提交的快照进行检查 | 服务端钩子 |
| update      |              为每一个准备更新的分支各运行一次              |                                                                                                                                                                             | 服务端钩子 |
| post-recive |                   整个提交过程完成后运行                   |                                      一般用于通知之类 更新其他系统服务或者通知用户 包括给某个邮件列表发信，通知持续集成（CI）的服务器                                       | 服务端钩子 |

参考地址：  
1: [git commit 、CHANGELOG 和版本发布的标准自动化](https://zhuanlan.zhihu.com/p/51894196)  
2: [用 husky 和 lint-staged 构建超溜的代码检查工作流](https://zhuanlan.zhihu.com/p/27094880)  
3: [掘金社区](https://juejin.im/post/5d27f84a6fb9a07ed064ddf1)  
4: [git 钩子](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90)