# React单元测试入门

## 软件测试中的测试种类
- 按阶段划分：单元测试、集成测试、系统测试、验收测试、(回归测试)      
- 按是否查看代码划分：黑盒测试、白盒测试、灰盒测试     
- 按手工/程序划分：手工测试、自动化测试    

## 开发模式
- TDD 测试驱动开发 Testing Driven Developemnt    
这是一种开发范式，即以测试来驱动整个项目，先根据接口或者需求来编写测试用例，然后再开发功能以满足所有的测试用例。

- BDD 行为驱动开发  Behavior Driven Development    
这是一种写测试的风格，是TDD的延伸，即测试写的需要像自然语言，或者就是自然语言，这样所有的协作方都可以理解和定义测试用例。鼓励开发者、测试人员、非技术人员的协作。 `shouldjs` 即是该风格的库。

## 单元测试必要性
单元测试是开发人员编写的、用于检测在特定条件下目标代码正确性的代码。是否存在必要性来源于判断单元测试到底带来了什么好处？
- 最大程度确保重构时稳定性
- 反向推动程序设计能力，将代码设计进行解耦
- 起到`README`功能
- 方便回归，无需人工手动校验，缩短问题反馈周期(开发阶段即发现)
- 发布更有信心 

## React项目测试可能用到的lib
- `react-dom/test-utils` (react-dom提供的测试工具集，react-dom存在的地方就存在，普遍用于各种react的测试库，组件提供)
- `react-test-renderer` (将react组件转换成纯JS对象，官方提供)
- `enzyme`
- `react-testing-library`
- `jest` 
- `jasmine`
- `mocha`
- `puppeteer`
- `selenium`
- `cypress`
- `karma`


## 其他常用项目他们是用哪些lib做的测试？
我们经常使用`react-router`、`redux`、`mobx`等，那他们的开发者在开发库的本身也会进行库进行测试。以下列举了他们测试自身时用的测试lib：

| 生态项目 | 测试当前项目时用到的lib |
| --- | ----| 
| `react-router` | `jest + react-test-renderer` |
| `react-redux` | `jest + testing-library/react` 由 `enzyme`迁移而来 |
| `mobx-react` | `jest + testing-library/react` 由`enzyme` 迁移而来 |
| `antd` |  `jest + enzyme + react-test-renderer` |
| `material-UI` |  `mocha + enzyme + react-test-renderer & karma` |
| `react-bootstrap` | `mocha + sinon + enzyme & karma` | 

注：
- `test-library`在UI库中使用的较少可能因为其出现的较晚，另外UI库测试侧重点需要看不同浏览器的状况，使用了`karma` 运行器，同时该运行器与`jest`集成起来需要解决的问题较多。

## 单元测试框架 jest / mocha / jasmine 选择
- `jasmine`  开箱即用、社区成熟、比较老
- `mocha`  需要较多配置适应新项目，社区成熟
- `jest`  开箱即用、API简单、社区较新，在React项目中用的较多   

## React测试类库 react-testing-library / enzyme 选择
以下摘抄了官方对于两个测试库的定义。 
- `enzyme`

    Enzyme is a JavaScript Testing utility for React that makes it easier to test your React Components' output. You can also manipulate, traverse, and in some ways simulate runtime given the output.

**简单总结：enzyme接口较多，可以操作组件来对state、props等进行精准测试**

- `react-testing-library` 

    - If it relates to rendering components, then it should deal with DOM nodes rather than component instances, and it should not encourage dealing with component instances.
    - It should be generally useful for testing the application components in the way the user would use it. We are making some trade-offs here because we're using a computer and often a simulated browser environment, but in general, utilities should encourage tests that use the components the way they're intended to be used.
    - Utility implementations and APIs should be simple and flexible.

**简单总结：接口简单，理念简单**


就前端单元测试而言，`react-testing-library `的已经足够，而在`enzyme`中可能会陷入过于关注内部实现的状态，深入组件进行操纵来符合我们的测试预期，脱离了用户行为而进行测试，这是一种很糟糕或者无效的测试。而 `react-testing-library` 仅提供dom节点的用户交互操作，并不提供针对组件实例的操作。当然`enzyme`同样可以遵循`react-testing-library`的理念来进行测试编写。 


## 使用 jest & react-testing-library 进行React单元测试

### jest 测试脚本的编写

#### 一个测试用例的基本组成部分

一个测试脚本文件由一个或者一系列测试用例来组成。
- `beforeAll(fn, timeout)`    
    当前测试文件开始测试前，支持返回promise，只执行一次
- `afterAll(fn, timeout)`    
    当前测试文件测试完成后，支持返回promise，只执行一次
- `beforeEach(fn, timeout)`    
    每个测试case开始前，执行多次，跟实际运行的case的数量相同，支持返回promise
- `afterEach(fn, timeout)`    
    每个测试case结束后，执行次数跟运行case的数量一致，支持返回promise
- `describe(name, fn)/describe.skip/describe.only`    
    一组测试case的集合，可以将关联性强的测试放在一起    
    skip 跳过；only 只运行当前的
- `describe.each(table)(name, fn)`    
    用于不同的数据运行相同的测试过程，可以将测试数据及结果放在数组中，作为table参数

        describe.each([
            [1, 1, 2],
            [1, 2, 3],
            [2, 1, 3],
        ])('.add(%i, %i)', (a, b, expected) => {
            test(`returns ${expected}`, () => {
                expect(a + b).toBe(expected);
            });

            test(`returned value not be greater than ${expected}`, () => {
                expect(a + b).not.toBeGreaterThan(expected);
            });

            test(`returned value not be less than ${expected}`, () => {
                expect(a + b).not.toBeLessThan(expected);
            });
        });

- `test(name, fn, timeout)/test.only/test.skip/test.todo`    
    单个的测试case    
    only 只运行当前的；skip跳过；todo待编写；

- 断言 `assertions`
    - 判断值    

            toBe            // 相当于 === 判断
            toEqual         // 值判断，不做引用判断  
            toBeDefined | toBeNull | toBeUndefined |  toBeNaN 
            toBeFalsy       // 针对 false, 0, '', null, undefined, NaN 判断
            toBeTruthy      // 非false
            toBeCloseTo     // 浮点数字判断
            toBeGreaterThan | toBeGreaterThanOrEqual | toBeLessThan | toBeLessThanOrEqual // 大小判断
    - 函数类判断     
    
            expect.toHaveBeenCalled         // 被调用 
            expect.toHaveBeenCalledTimes    // 被调用的次数
            expect.toHaveBeenCalledWith | expect.toHaveBeenLastCalledWith  // 测试被调用的数据 
            expect.toHaveReturned | expect.toHaveReturnedTimes | expect.toHaveReturnedWith | expect.toHaveLastReturnedWith | expect.toHaveNthReturnedWith  // 测试返回的数据、次数、是否返回了
    - 属性类校验      

            toHaveLength(number)        // 检测.length属性值
            toHaveProperty(key, value?) // 检测是否有改属性或属性对应的值

    - 模糊判断  

            toMatch(regexpOrString)     // 进行正则匹配   expect('xx').toMatch('x');
            toMatchObject(object)       // 检测是否是其子集
            expect.anything()           // 可以匹配除了 null, undefined的情况   expect('tiger').toEqual(expect.anything());
            expect.any(constructor);    // 匹配是否由construcotr产生  expect(11).toEqual(expect.any(Number));
            expect.arrayContaining | expect.objectContaining | expect.stringContaining   // 测试包含
            expect.stringMatching(string | regexp) // 字符串匹配
            toBeInstanceOf(Class)       // 测试实例instanceof expect(new CompA()).toBeInstanceOf(CompA);
            toContain | toContainEqual  // 检测是否包含，针对数字或者string 
    - promise及错误相关    
            
            resolves            // expect(Promise.resolve('lemon')).resolves.toBe('lemon');
            rejects & toThrow   // expect(Promise.reject(new Error('octopus'))).rejects.toThrow('octopus');
    - 快照类     
    
            toMatchSnapshot | toMatchInlineSnapshot() // 进行快照对比

    - 断言数量   

            expect.assertions(number)   // 验证当前测试中执行了多少个断言，比如说在回调中有断言判断
            expect.hasAssertions        // 当前测试用例中至少执行了一次断言判断


举个可能的例子：

    // 文件 utils.test.ts

    beforeAll(() => {
        // 做全局的init，只针对当前文件
    }))
    beforeEach(() => {
        // 做每个case的Init
    })
    afterEach(() => {
        // 每个case的cleanup 
    })
    afterAll(() => {
        // 全局的cleanup
    })

    describe('group one', () => {
        test('demo 1', () => {
            expect.assertions(10); // 执行了10次断言判断
            expect(1).toBe(1);  // ===
            expect(1).toEqual(1);
            expect({ a: 1 }).toEqual({ a: 1 })
            expect({ a: 1, b: 1 }).toEqual({  })
            expect([1,2,3]).toHaveLength(1);
            expect({ a: 1}).toHaveProperty('a') / toHaveProperty('a', 1);
            expect(pressSpy).toHaveBeenCalledWith(expect.objectContaining({
                x: expect.any(Number),
                y: expect.anything()
            }));
            expect({ a: 1, b:1 }).toMatchObject({ a: 1 })
            const changeSpy = jest.fn();
            // changeSpy.mock.calls
        });

        test.only('demo 2', () => {

        });

        test.skip('demo 3', () => {

        });

        test.todo('demo 4');
    });

    test('demo 11', () => {});

    test.only('demo 12', () => {});

    test.skip('demo 13', () => {});

    test.todo('demo 14');

    describe.only('group two', () => { })

    describe.skip('group three', () => { });


#### 支持mock
- 使用 `jest.fn`      
`jest.fn()`产生的mock函数，注入了一些用于处理mock的方法或者属性.

        const add = jest.fn();
        add.mock.calls   // 调用的参数集合
        add.mock.results   // 返回的结果集合
        add.mockReset()  // 清空调用、返回的情况
        add.mockImplementation(fn) // mock一个新的函数实现
            add.mockImplementation((m, n) => m * n);
        add.mockImplementationOnce(fn) // mock一次函数实现
        add.mockReturnValue(value)  // mock返回值
        add.mockReturnValueOnce(value) // mock一次返回值


- 使用 `__mocks__` 目录
- 时间的mock(针对`settimout`, `setInterval`等)     
    `jest.useFakeTimers()` 打开time mock 功能       
    `jest.runAllTimers`      
    `jest.runOnlyPendingTimers` (递归时)    
    `jest.clearAllTimers`  

#### 异步问题 
- test done支持    

        test('xxxx', done => {
            doCallback((err, rets) => {
                // done();  // 正确的情况
                // done('error')  // 错误的情况
            })
        })

- 支持 async/await
    
        test('test 1', async () => {
            const rets = await fetchData();
            // expect(...)
        });

- 支持返回promise

        test('test promise', () => {
            return fetchData().then(data => {
                expect(data).toBe('xxx');
            })
        })

        test('test promise fail', () => {
            expect.assertions(1); // 重要
            return fetchData().catch((e) => {
                expect(e).toMatch('error')
            })
        })

#### 覆盖率
`jest` 内置了 `istanbul` 输出的覆盖率结果，包含了4个部分：
- statement coverage (语句覆盖率)
- branch coverage (分支覆盖率)
- function coverage (函数覆盖率)
- line coverage (行覆盖率)
可以根据需要设置覆盖率要求


### react-testing-library 提供的方法有：
#### 异步方法：   
- `waitFor(callback, options): Promise<T>`   
        注：等待callback的所有断言都成功运
        
- `waitForElementToBeRemoved`    

    // 例子：
    await waitFor(() => expect(getByText('xxx')).toBeInDocument());

#### dom节点 搜索方法
- 查询前缀：`get/query/find`    
    `getBy` `getAllBy`  [如果未找到，则会报错]    
    `queryBy`  `queryAllBy`  [未找到，则返回null]    
    `findBy` `findAllBy`   [返回promise，可以理解成`getBy*` + `waitFor`]

- 查询的目标:    
    `ByLabelText` （根据label标签）    
    `ByPlaceholderText` (placeholder内容)    
    `ByText` (根据text node)     
    `ByAltText` (根据alt属性)    
    `ByTitle` (根据title属性)    
    `ByDisplayValue` (根据表单元素展示的值)    
    `ByRole` (根据W3C规定的role属性)    
    `ByTestId` (根据设置的data-testid属性)     

        例子：
            const inputEle = await findByPlaceholderText(container, '请输入邮箱'); 
            const inputEle2 = getByRole(container, 'textbox'); // input or textarea
            const inputEle3 = getByDisplayValue(container, 'ss@itiger.com'); // 根据表单元素呈现的值查询

- 同时也支持 dom接口 querySelector, querySelectorAll

#### 事件方法 fireEvent
- 使用方法1：`fireEvent(node, event)`;  
- 使用方法2：`fireEvent[event_type](node, eventProperties)`;

        例子:
        fireEvent(node, new MouseEvent('click', { bubbles: true, cancelable: true }));

        fireEvent.change(emailNode, { target: { value: 'x.x@itiger.com' } });

        // 按下Enter键
        fireEvent.keyDown(domNode, { key: 'Enter', code: 13 })

        // 按下字母A
        fireEvent.keyDown(domNode, { key: 'A', code: 65, charCode: 65 })

还提供了 `createEvent` 来构造`event`事件。(当我们需要去获取event的一些属性时)

        const myEvent = createEvent.keyDown(node, { key: 'A', code: 65, charCode: 65 })
        fireEvent(node, myEvent)

额外有个 https://github.com/testing-library/user-event 将经常用到的事件操作做了封装。


#### 调试能力：即 debug 方法，打印当前的dom结构
`screen.debug`


#### render 方法
    const { container, debug, rerender, asFragment, unmount, ...querys } = render(<MyComponent />, { container, ... })
    
该方法默认会创建个div节点，然后append到`document.body`上，可以通过container改变到自己想要挂载的节点上。    
    返回值包含了上述的`find/get/query`等，以及`container`、`asFragment`方法（返回该组件渲染后的DocumentFragment)， `rerender`方法，`umount`方法，以及`debug`调试方法

注：
- `rerender` 在需要测试组件`props`改变时，组件里有对应的处理逻辑时常会用到。

        例子如：
        const { rerender } = render(<User id={1} />);
        // id=2时重新render
        rerender(<User id={2} />)

- `asFragment`常在进行快照对比时用到

- `cleanup`    
    在afterEach时自动unmount，只要支持afterEach的测试库不需要显示调用
- `act`    
    对`react-dom/test-utils`的 act 函数的封装
    任何页面上、数据上、行为上的导致组件的更新，都应放置到act里。此方法为了组件的处理更接近React在真实的浏览器上。

#### 其他库增强的能力：
- `jest-dom`    
增加了dom相关的一些断言判断功能，如是否有class, 是否disabled, 包含 html，是否包含属性 
    对 expect 增加了 `toBeDisabled | toBeEnabled | toBeEmpty | toBeInTheDocument | toBeInvalid | toBeRequired | toBeVisible | toHaveClass | toHaveFocus | toHaveStyle | toHaveFormValues | toHaveValue | toBeChecked | toHaveAttribute | toHaveTextContent | toContainHTML | toContainElement` 

- `userEvent`    
`userEvent`是将`fireEvent`部分操作封装成一些方法，使用起来更简便。目前支持如下：

        userEvent.type(node, 'x'); // 输入x
        userEvent.click(node); // 点击
        userEvent.dblClick(node); // 双击
        userEvent.selectOptions(node, '86'); // 选择select的值
        userEvent.tab();  // 键盘tab切换

## tiger-new中使用
    
### 新建项目
    tiger-new tiger-new-demo
    npm run test

### 编写测试用例
请看项目[demo/tiger-new-demo]

- 案例：普通组件测试 `components/forms/Email`
- 案例：mock文件  `components/forms/CountrySelect`
- 案例：mock请求、异步支持  `components/forms/CountrySelect`
- 案例：time的mock   `components/forms/Counter`
- 案例：直接mock  `components/forms/Comment`
- 案例：不mock，直接赋值  `components/forms/Footer`
- 案例：快照  `components/forms/Agreement`
- 案例：导入时将环境分离 `components/forms/Slider`
- 案例：导入变量和函数的mock `components/forms/Language`

### 测试覆盖率

`npm test -- --coverage --watchAll=false`


## 推荐的实践方式
- 测试描述应当是表达力极强的，错误时则很快能够进行定位（更侧重业务方面定位）
- 单个测试用例应当是单条branch的进行的
- 测试用例的逻辑应当足够简单
- 测试用例的运行应当足够快，避免因依赖而耗时很多
- 不应过多关注内部实现，从用户操作角度考虑出发即可。（观察到的只有dom）
- 如果组件内有分拆成多个内部组件，且这些只被用于于当前组件内，则不需要额外写测试，直接测试当前组件即可覆盖。
- 直接以组件或者模块命名即 `CompA.test.tsx`，如果有多个需要测试，可以考虑`__tests__` 目录。
- 使用`describe`、`test`即可，不建议使用别名`fdescribe`, `xdescribe`, `xit`, `fit`
- 组件按照业务来进行多个分支测试，且每个分支测试有多个case需要运行到，可以使用`describe`来区分。非必要情况直接用`test`即可。
- 快照对比，多数项目场景是不必要的
- 不可侵入组件实例内部来编写测试，与真正人工操作违背
- 就近mock。如果只是编写单测，那么就近mock较便捷。当开始集成测试时，可能就近mock时较麻烦，可以改成统一的全局mock。
- 组件的交互对象有：人、`server/API`、其他关联组件、(`formutil`)等，所以测试时这些输入输出也应该考虑到
- 覆盖率可以根据时间和业务来逐步提升，不用严格100%
- 工具包的测试，使用参数化测试更高效




#### 参考资料
- [可访问性](https://www.w3cplus.com/wai-aria/web-accessibility.html)
- [aria-query](https://github.com/A11yance/aria-query#elements-to-roles)
- [软件测试概念](https://www.cnblogs.com/findyou/p/6480411.html)
- [单元测试](https://www.jianshu.com/p/dad49f7c78d3)
- [react-test-rerender](https://reactjs.org/docs/test-renderer.html)
- [react-test-utils](https://reactjs.org/docs/test-utils.html)
- [React](https://github.com/facebook/react)
- [react-router](https://github.com/ReactTraining/react-router)
- [mobx-react](https://github.com/mobxjs/mobx/mobx-react)
- [antd](https://github.com/ant-design/ant-design/)
- [material-ui](https://github.com/mui-org/material-ui)
- [cypress](https://www.cypress.io/)
- [react-bootstrap](https://github.com/react-bootstrap/react-bootstrap)
- [mocha](https://mochajs.org/)
- [前端测试框架对比](https://www.cnblogs.com/lihuanqing/p/8533552.html)
- [jest-dom](https://github.com/testing-library/jest-dom)
- [BDD](http://cuketest.com/zh-cn/bdd/why_bdd.html)
- [shouldjs](https://github.com/shouldjs/should.js)