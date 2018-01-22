---
title: 代码测试覆盖率分析
date: 2017-08-20
tag:
- 前端
- 测试
- test
- coverage
---

## 背景

最近我们前端团队在重构大量的 UI 组件，为了保证代码质量，我要求团队中的成员必须编写单元测试，并且测试覆盖率达到 80% 以上。那么问题来了，为什么是 80% 的覆盖率？ 这是一个硬性的考核指标吗？

> 这里所说的测试覆盖率，是指的是开发人员写的单元测试的覆盖率，不是测试人员的功能测试的覆盖率。


## 哪些地方需要写单元测试？

为什么需要写单元测试就不再阐述，我相信大家都知道，特别是在持续集成过程中的重要性。但是，从我的经历来看，当前的软件市场环境中，不管是用的瀑布模式，还是螺旋模式，还是敏捷模式，很多软件没有写单元测试。

我也是一个程序员，每天需要写一些的业务代码，对于写单元测试来说，确实需要我很多时间和精力，因为它也需要设计用例和一些体力活。所以在我们的一些项目中也存在很多功能没有单元测试，主要原因有以下几个点:

- 业务逻辑更新太快，单元测试不可复用；
- 业务时间紧急，迭代周期时间短，没有时间写单元测试；
- UI 上很多测试，通过单元测试代码无法覆盖。

在《软件测试》一书中讲测试的原则，第一条就是：“完全测试程序是不可能的”。所以对于以上部分需不需测试，取决于你软件性质，时间和团队。但是对于满足以下几点代码我建议需要编写单元测试：

- 和安全相关的代码逻辑；
- 核心的功能模块，函数；
- 短期不会发生变化的 UI 组件；
- 提供外部调用的接口。

<!-- more -->

## 测试覆盖率报告


如果完全通过测试覆盖作为质量标准是存在问题的，我们在检查一个测试覆盖了的时候往往会通过一些工具去检查，程序员是可以通过一些方式让数字看上去漂亮，但是这没有意义。我们应该把它作为一种发现未被测试覆盖的代码的手段，同时也是一种学习的手段，为什么这段代码没有覆盖到？ 如果这个函数的参数发生了变化会怎么样？ 这段代码逻辑怎么这么复杂？

通过分析未被测试覆盖的代码，找到是设计问题，还功能理解有问题，还是说着就是一段废代码，它可以帮助开发者能够更好的理解背后的事情，可以检查程序中的废代码，然后在以后的设计中做很好的抽象，做可测试的代码。


各种开发语言都有对应的测试框架，可以生成测试报告，在本文中我以前端的 javascript 为示例， `karma` + `istanbul` 工具生成报告。

- `karma` 是一个测试框架；
- `istanbul` 是 JavaScript 程序的代码覆盖率工具。

怎么生成测试报告这里就不讲，有很多教程，也可以查看官方文档 [istanbul](https://github.com/gotwarlost/istanbul)。这里我们先来看一下生成出来的测试报告。 以下是 [rsuite](https://github.com/rsuite/rsuite) `src/utils` 目录下文件的测试报告, 这是打开的一个生成 `html` 格式的测试报告：

{% asset_img 1.png RSUITE 测试覆盖率 %}

从图中我们可以看到它有四个指标：
- Statements: 语句覆盖率，执行到每个语句；
- Branches: 分支覆盖率，执行到每个if代码块；
- Functions: 函数覆盖率，调用到程式中的每一个函数；
- Lines: 行覆盖率, 执行到程序中的每一行。

> 每一个指标都列出了覆盖的比例和数量情况，其中 `Statements` 与 `Lines` 比例和数量是一致的，那它们有什么不同呢？
> 在代码中往往存在一些书写不规范的情况，比如一行多个语句，这个时候它们统计的覆盖率就会有差异。 这里又有一个值得思考的问题就是，代码覆盖率工具是怎么统计一行多个语句这种代码的？ 后面讲到统计原理的时候会讲到。



另外，我们通过图中可以看出  `decorate.js` 这个文件相对来说测试覆盖率较低，我们进入再具体分析一下，在那些地方没有覆盖到：

{% asset_img 2.png decorate.js 测试覆盖率 %}

从图中我们可以看到红色部分和黄色, 都是在测试用例中没有覆盖到的地方:

- `getProps` 函数，该函数式 `export` 出去的一个函数，但是在测试用例中没有覆盖到；
- `typeof size === 'object'` 代码块没有覆盖到；
- `Component.propTypes={}`.. 这里黄色部分，是一个默认值设置，说明这个默认值一直没有被使用过；


在图中左侧，显示行号的地方有一个 `12x`、`9x`、`4x`，这个代表了该行语句被执行的次数， 通过这个清晰的报告，我们可以在代码中看出那些函数，那些代码块没有被执行，从而去分析原因，修正测试用例，完善代码逻辑，提高质量。

## 生成测试报表原理

我先来看一下 `istanbul` 生成的测试报告中有个 `lcov.info` 文件, 这里我只贴出关于 `decorate.js` 文件这部分的内容:

```
SF:/Users/simonguo/workspace/rsuite/src/utils/decorate.js
FN:25,getClassNames
FN:39,getProps
FN:41,(anonymous_2)
FN:50,decorate
FN:51,(anonymous_4)
FNF:5
FNH:3
FNDA:237,getClassNames
FNDA:0,getProps
FNDA:0,(anonymous_2)
FNDA:12,decorate
FNDA:12,(anonymous_4)
DA:4,1
DA:11,1
DA:18,1
DA:27,237
DA:28,237
DA:30,237
DA:32,237
DA:40,0
DA:41,0
DA:42,0
DA:44,0
DA:51,12
DA:52,12
DA:53,12
DA:54,12
DA:56,12
...
```
`FN` 代表函数，
`25`,`39`,`41`,`50`,`51` 这些行分布对应源代码中的函数开始的行号，
`FNF:5` 代表一共有5个函数
`FNH:3` 其实 3 个函数被测试所覆盖，
`FNDA:237,getClassNames` 代表了 `getClassNames` 这个函数被执行了 237 次。
...

等等，在文件中详细记载了行号，以及代码的执行情况，大家可以再对照前面的那张“测试覆盖率”图片进行分析，可以详细的看出整个 `lcov.info` 文件中记录内容。有了这样一份记录信息就能够生成出一份可视化的测试报告，也可以上传到 [coveralls](https://coveralls.io/)，展示给大家。 那么这里需要思考的问题是，这样一份数据统计记录是怎么统计出来的呢？


> 如果希望有些代码被忽略，不进入覆盖统计，istanbul 提供注释语法 ，查看[Ignoring code for coverage purposes](https://github.com/gotwarlost/istanbul/blob/master/ignoring-code-for-coverage.md)


javascript 覆盖率统计的核心思想，是在源代码相应的位置注入设定的统计代码，当执行测试代码的时候，代码运行到注入的地方，就会执行对应的统计代码，生成覆盖率统计报告。大概步骤如下：

- 第一步：生成语法树，对源代码进行语法分析，解析，然后生成语法树。
> 生成出来的结构如下，这段代码来自 [`esprima`](http://esprima.org/)， A simple example on Node.js REPL:
```js
> var esprima = require('esprima');
> var program = 'const answer = 42';

> esprima.tokenize(program);
[ { type: 'Keyword', value: 'const' },
  { type: 'Identifier', value: 'answer' },
  { type: 'Punctuator', value: '=' },
  { type: 'Numeric', value: '42' } ]

> esprima.parse(program);
{ type: 'Program',
  body:
   [ { type: 'VariableDeclaration',
       declarations: [Object],
       kind: 'const' } ],
  sourceType: 'script' }
```

- 第二步：注入统计代码，在语法树相应的位置注入统计代码，在程序执行到这个位置的时候对相应的全局变量赋值，确保执行之后能够根据全局变量知道代码的执行流程。到这里就解决了前面说的“一行如果有多个语句怎么统计？”的问题。
- 第三步：再把注入统计代码的语法树，生成对应的 javascript 代码。
> 以下是 [`escodegen`](https://github.com/estools/escodegen) 的一段示例代码
```js
// A simple example: the program

escodegen.generate({
    type: 'BinaryExpression',
    operator: '+',
    left: { type: 'Literal', value: 40 },
    right: { type: 'Literal', value: 2 }
});

// produces the string '40 + 2'.
```


- 第四步：将生成好的 javascript 代码交给执行环境（nodejs或者浏览器）运行。
- 第五步：执行单元测试，产生的统计信息，放到全局标量中。
- 第六步：根据全局标量中的覆盖率信息生成特定格式的报告，这样我们就看到了 `lcov.info` 文件和 `.html` 文件。

这个步骤是依据 `istanbul` 统计 javasript 的原理，其他语言的一些统计工具没有接触过，但是基本的思想应该都是大同小异的。在 javasript 对语法分析，生产语法树再还原 javasript 代码是有一些开源工具的，所以如果有兴趣的童鞋要自己实现一套代码覆盖率的功能，只需要写好注入的统计代码逻辑和运行环境的处理。


## 总结

对一个持续集成的项目来说，单元测试非常重要，同时最好具有较高的测试覆盖率。再次强调测试覆盖率是一种发现未被测试覆盖的代码的手段，它不是一个考核质量的目标。

另外，我们维护的开源项目 [rsuite](https://rsuitejs.com) ,是一套 React 的 UI 组件库，如果你对此感兴趣，或者使用中遇到任何问题，可以联系我们 [Discord: join chat](https://discord.gg/GmPXTH3)


> 本文作者：[郭小铭](https://github.com/simonguo)