
## 面向B端金融系统前端架构的思考

有幸接触到面向B端金融的系统（暂且叫Bitbal吧），开发近2年以来，思考及开发等方式与C端差别很大。    
项目背景：系统主要服务于券商，主要是用于股票等交易后的清算，其对账、报表等是核心。 以下说的文件可在[Demo](https://github.com/alphalion-tool/react-crm-kit)找到。

之所以开发这个系统，是想替换掉原有的系统。原有的系统体验大概可以回溯至少10年前，与现代的网页体验差别较大。所以原有系统升级起来极为缓慢，原系统后端使用的是一种小的开发语种，可能在多数的榜单中都看不到，前端也比较古老。    

近些年前端的工程化发展很迅速。React, Angular, Vue倍受推崇，这些极大的促进了前端工程化的开发效率。而Bitbal共拥有近200个模块（目前为止），工程化很是重要，选择React作为支持的基础框架，模块化的开发体验上了个台阶。    


### 基础组件、框架，优先    
架构时，路由，状态管理等，以及对应的开发方式，这些需要尽快明确下来，否则开发无从下手。Bitbal采用的是`react-router`中`browser router`来进行路由管理的。数据、状态管理则用使用的是`redux`。    

开发时，基础框架和组件是必需品。这些可以是团队内自己维护的组件，也可以是第三方的。如果团队小，可以选择第三方的。如果自身够强大，维护自己的一套也是个好的选择。但B端产品，考虑到交互给客户，所以选择组件、代码库等方面要考虑授权协议是否满足需要。Bitbal在梳理出开源的时候，将内部开发的组件替换成`antd`。现在来看，`antd`是个很优秀的UI组件库。然后产品开发之初，`antd`还没那么强大。当然现在`antd`在表格的展示处理上似乎还是有些问题。[FixedDataTable](http://schrodinger.github.io/fixed-data-table-2/)是个不错的可选择的表格组件。    


### 前后端分离
后端是基于`Java Spring boot`框架开发。最初，前端是与后台的项目混在一起，前端开发的时候需要搭建整个Java项目，这种体验是很糟糕的。这就要求，每位前端开发都会涉及到后台的一些命令等，浪费了很多属于开发的时间。前后端彻底分离变得极为迫切。 

分离时，前后端约定一个文件路径，该路径存储了前端编译后的模板和静态文件。前端按照约定的路径部署，后端则从约定的路径读取模板以及静态文件。

前端提供一个`index.pug`模板作为系统的入口。后端读取该文件，并将需要的数据注入到pug模板里最终渲染给用户。    
静态文件也是路由到约定的路径中。该路径的模板和静态文件依赖于`webpack`打包后产生的文件。    
前端开发时，则使用`webpack-dev-server`来进行静态文件以及数据请求的处理。数据请求通过proxy的方式去获取后端提供的接口。

        
        约定的目录结构：

        build
            ├─ static 
            |    └─ js/css/images/fonts/...
            |
            ├─ templates
            |    └─ index.pug


        webpack-dev-server做的事：

            静态文件  ---->  webpack-dev-server   ---> 读取src下的文件
            api请求  ---->  webpack-dev-server  ---> 代理转发至后台server获取数据并返回

开发完成，进行`npm run build`，将最新产生的文件放置在`build/static`中，而模板里面的文件MD5戳在编译的过程中也是需要修改的。    
分离后，前端不依赖于后端的项目开发、部署，完全可以将api请求proxy至`mock server`。 

该功能见`webpack-dev-server/server.js`


### 人人有风格，但规范先行
团队规模一直在扩大，需要有一个良好的规范。从`js`，`css`到`git flow`及`git commit msg`等，都是需要规范的一步。    

- 对于代码规范，为了更好的执行，校验应该在代码提交前，这样才能保证提交至仓库的不会有不符合规范的。使用`git commit hook`将`eslint`, `csslint`, `react flow`在`commit`前执行，从而使代码基础质量有所提升。    
- 对于流程规范，在规模大、需求并行等项目极为重要。Bitbal项目需要部署到客户或者我们自己提供的服务器上。且客户对频繁部署有排斥心理。每次迭代需要数周或者数月。`git flow`的使用，将版本管理的有条不紊，区别对待dev, test, trunk, master, hotfix等情况。    
- `Code Review`这个过程很关键，任何团队都有初级、每个人也都有不擅长的东西。在review过程中，可以避免一些bug，以及提升新人的开发水平。这个过程我们是在merge request时做的。      
- `git commit`时信息也需要做规范。规范后，在git log时也很清晰，同时也可以根据工具快速梳理出近期做的模块、涉及到的模块。     
- `merge request`时，提供的描述要更能理解。根据`Feature`, `Bug Fixes`, `Test`进行分类，实际情况可以增加更多的分类。

这部分可参考[frontend-stand](https://github.com/alphalion-tool/frontend-standard)

### 目录结构设计
目录结构的合理清晰会让开发起来更简单，更易管理。Bitbal采用路由的方式来划分模块。    
    

    目录结构如下：    
    
    ├─ actions      // 需要的基础actions
    ├─ assets       // 资源类，图片、字体
    |     ├─ images     // 图片资源
    |     ├─ fonts      // 字体资源
    |     ├─ locales    // 多语言文件
    |     └─ style      // 样式
    ├─ components   // 公用的组件
    ├─ config       // 配置信息，如常量，导航配置等
    ├─ containers   // 全局容器
    ├─ polyfills    // polyfill
    ├─ reducers     // 需要的基础reducer，与最开始的actions对应
    ├─ routes       // 每个路由对应的模块，均放置在该目录
    ├─ schemas      // 每个数据实体对应的schemas
    ├─ services     // 网络请求管理
    ├─ store        // redux store管理
    ├─ style        // 样式
    ├─ utils        // 工具类，如格式化等
    └─ index.js     // 入口


    在上述的routes目录中可以查看单个route的写法，而单个route的内部目录如下：
    
    AccountList
        ├─ components       // 内部routes用到的公用的，或者自己拆解的
        |       ├─ AccountListQuery.js
        |       ├─ AccountListQuery.scss
        |       ├─ AccountSelect.js
        |       └─ AccountSelect.scss
        |
        ├─ config           // 配置，如表单、表格等
        |       ├─ table.js
        |       ├─ form.js
        |       └─ sheet.js
        |
        ├─ containers       // 容器
        |       ├─ AccountList.js
        |       └─ AccountList.scss 
        |
        ├─ modules          // action, reducer集合
        |       ├─ action.js
        |       └─ reducer.js
        |         
        ├─ routes           // 子路由
        └─ index.js         // 入口



### 多语言，绕不过去的坎
多语言在多数国际化的项目中遇到的越来越多了。Bitbal项目是服务于北美、亚太地区的。所以多语言是需要的。    
前端根据`react-intl`来实现多语言。将多语言文件放置在一个目录中，然后按语种进行细分。然后根据业务进行细分并创建对应的多语言文件。
见Bitbal项目中的`src/assets/locales`，同时提供一个装饰器用于在组件中使用，见`src/assets/locales/index.js`中的`intlInject`。

        locales
            ├─ en
            |   ├─ common.js    // 公共的，如按钮名称，全局都用到的
            |   ├─ nav.js       // 导航用到的
            |   └─ *.js         // 具体业务用到的，按业务名称划分。
            |
            ├─ zh
            |   ├─ common.js
            |   ├─ nav.js
            |   └─ *.js


### 开发效率提升
#### webpack性能
webpack性能之路，一直都是在做。涉及到dev-server，build的时候。当模块较多时，`npm run dev`需要将近200s才能最终运行起来。从多进程以及dll的思路入手。`happy-pack`和`dll`可以有效的提升效率。

这部分见`webpack.common.js`，`webpack.dll.js`

### 数据、状态管理
数据状态管理用的是`redux`。初始加载只加载部分必须的。而模块的reducer则只当模块被加载时，才通过`injectReducer`将该模块对应的reducer注入到原有的store中。     
为了防止action type命名冲突问题，针对每个文件的action的type做了处理，每个文件被注入都是有一定顺序的，按照顺序增加index。在名称中也包含这些index。如在实际应用中可以看到action type为`APP-0-switchLang`，`AUTH-1-login`，`AUTH-1-logout`等。     

这部分参考`src/actions`，`src/reducers`，`src/store`

### 前端优化
前端优化是个大课题，从React渲染到最终打包的文件不一而足。    

诸如：

- 从React角度，减小无效渲染，会用`shouldCompentUpdate`与`immutable`，记忆函数也能在很多场景中有效减小无用计算
- 从模块化角度，将页面分块，提出更多通用的组件。
- 从加载角度，模块异步加载十分重要`react-loable`与`react-router`合理搭配。减小不必要的首次加载
- 打包时，考虑image、js、css的压缩等优化处理
- ...


### 测试，控质利器
以前的开发多数是不写测试，尤其是面向C端的业务，业务不长久，可能一点动力都没有。B端业务，尤其是该项目，持久性很强。测试是回避不了的。当然实际开发中，面对需求频繁调整，功能开发和测试也会频繁修改。但这些也不是不写测试的理由。    
测试从粒度上看，最大的是模块的测试。小的则是单个组件、函数测试（单测）。单测相对容易些。    
模块的测试，可能需要网络IO的要求，这些可以通过拦截用本地mock数据替代。    
覆盖率方，需要结合团队的情况来设置，针对function, branch, line可以逐个调高覆盖率要求。

测试使用的`jasmine`，参考`jasmine.json`。

### 常量的传递
项目里遇到的常量有几百种类型。诸如各种type, currency, country等等。最初是硬编码在前端，而后发现，针对不同的客户，这个拓展性很困难。最后全部由server来控制。    
可以按照需要来加载，不过这样会增加测试的复杂性以及业务逻辑处理的复杂性。目前是在登录时预加载下来，然后注入到`window`里。既有的组件涉及到常量的修改起来便容易许多。


### API错误提示
错误提示，在网络请求处，进行统一处理。根据前后端约定的code来做不同的处理。    
当然也需要分类，是`Message`, `Confirm`, `Notification`，需要根据不同的情况来去分类。


### 前端错误信息收集
测试时，线上避免不了错误。对错误的跟踪，在排查问题的时候变得很有用。但在压缩后，如果没有sourcemap，很难去做定位。所以在错误的时候，由前端或者后端去加载sourcemap并将真实的位置信息记录下来。    
针对业务数据机密的情况，即使远程部署了，依旧可以通过导出日志来进行追踪并修复。

参考`src/utils/errorReport.js`

### 内部工具
结合内部的情况，开发一些供内部使用的工具。效率先行。能让程序做的事，就让程序来处理。    
比如说：    

- jenkins自动化测试，如测试、部署
- gitlab hook提示，merge request, commit等重要信息
- mock data server
- wiki平台（分享等）



