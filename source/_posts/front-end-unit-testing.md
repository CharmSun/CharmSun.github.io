---
title: 前端自动化单元测试
date: 2018-04-03 16:13:05
categories:
- 前端
tags: 
- 前端单元测试
img: /2018/04/03/front-end-unit-testing/brief.jpg
---

随着Web业务的日益复杂化和多元化，前端开发也有了前端工程化的概念，前端工程化成为目前前端架构中重要的一环，本质上也是软件工程的一种，因此我们需要从软件工程的角度来研究前端工程，而自动化测试则是软件工程中重要的一环。本文就研究一下前端领域中的自动化测试，以及如何实践。

## 什么是单元测试
> 单元测试（unit testing），是指对软件中的最小可测试单元进行检查和验证。对于单元测试中单元的含义，一般来说，要根据实际情况去判定其具体含义，如C语言中单元指一个函数，Java里单元指一个类，图形化的软件中可以指一个窗口或一个菜单等。总的来说，单元就是人为规定的最小的被测功能模块。单元测试是在软件开发过程中要进行的最低级别的测试活动，软件的独立单元将在与程序的其他部分相隔离的情况下进行测试。——百度百科

## 为何要测试
以前没有编写和维护测试用例的习惯，在项目的紧张开发周期中也没时间去做这个工作，相信有不少开发人员都不会重视单元测试这项工作。在真正写了一段时间基础组件后，才发现自动化测试有很多好处:
1. 提升代码质量。虽不能说百分百无bug，但至少说明测试用例覆盖到的场景是没有问题的。
2. 能快速反馈，能确定UI组件工作情况是否符合自己预期。
3. 开发者会更加信任自己的代码，也不会惧怕将代码交给别人维护。后人接手一段有测试用例的代码，修改起来也会更加从容。测试用例里非常清楚的阐释了开发者和使用者对于这段代码的期望和要求，也非常有利于代码的传承。

当然由于维护测试用例也是一大笔开销，还是要基于投入产出比来做单元测试。对于像基础组件、基础模型之类的不常变更且复用较多的部分，可以考虑写测试用例来保证质量，但对于迭代较快的业务逻辑及生存时间不长的部分就没必要浪费时间了。

因此github上看到的star较多的牛逼开源前端项目基本上都是有测试代码的，看来业界大牛们都是比较重视单元测试这块的。

## 相关概念
### TDD
TDD是Test Driven Development 的缩写，也就是测试驱动开发。

通常传统软件工程将测试描述为软件生命周期的一个环节，并且是在编码之后。但敏捷开发大师Kent Beck在2003年出版了 Test Driven Development By Example 一书，从而确立了测试驱动开发这个领域。

TDD需要遵循如下规则：
- 写一个单元测试去描述程序的一个方面。
- 运行它应该会失败，因为程序还缺少这个特性。
- 为这个程序添加一些尽可能简单的代码保证测试通过。
- 重构这部分代码，直到代码没有重复、代码责任清晰并且结构简单。
- 持续重复这样做，积累代码。

TDD具有很强的目的性，在直接结果的指导下开发生产代码，然后不断围绕这个目标去改进代码，其优势是高效和去冗余的。所以其特点应该是由需求得出测试，由测试代码得出生产代码。打个比方就像是自行车的两个轮子，虽然都是在向同一个方向转动，但是后轮是施力的，带动车子向前，而前轮是受力的，被向前的车子带动而转。

### BDD
所谓的BDD行为驱动开发，即Behaviour Driven Development，是一种新的敏捷开发方法。它更趋向于需求，需要共同利益者的参与，强调用户故事（User Story）和行为。2009年，在伦敦发表的“敏捷规格，BDD和极限测试交流”中，Dan North对BDD给出了如下定义：
> BDD是第二代的、由外及内的、基于拉(pull)的、多方利益相关者的(stakeholder)、多种可扩展的、高自动化的敏捷方法。它描述了一个交互循环，可以具有带有良好定义的输出（即工作中交付的结果）：已测试过的软件。

它对TDD的理念进行了扩展，在TDD中侧重点偏向开发，通过测试用例来规范约束开发者编写出质量更高、bug更少的代码。而BDD更加侧重设计，其要求在设计测试用例的时候对系统进行定义，倡导使用通用的语言将系统的行为描述出来，将系统设计和测试用例结合起来，从而以此为驱动进行开发工作。

大致过程：
1. 从业务的角度定义具体的，以及可衡量的目标

2. 找到一种可以达到设定目标的、对业务最重要的那些功能的方法

3. 然后像故事一样描述出一个个具体可执行的行为。其描述方法基于一些通用词汇，这些词汇具有准确无误的表达能力和一致的含义。例如，expect, should, assert

4. 寻找合适语言及方法，对行为进行实现

5. 测试人员检验产品运行结果是否符合预期行为。最大程度的交付出符合用户期望的产品，避免表达不一致带来的问题

### 覆盖率
如何衡量测试脚本的质量呢？其中一个参考指标就是代码覆盖率（coverage）。

什么是代码覆盖率？简而言之就是测试中运行到的代码占所有代码的比率。其中又可以分为行数覆盖率，分支覆盖率等。具体的含义不再细说，有兴趣的可以自行查阅资料。

虽然并不是说代码覆盖率越高，测试的脚本写得越好，但是代码覆盖率对撰写测试脚本还是有一定的指导意义的。



## 前端单测工具栈
### 测试框架
主要提供了清晰简明的语法来描述测试用例，以及对测试用例分组，测试框架会抓取到代码抛出的AssertionError，并增加一大堆附加信息，比如那个用例挂了，为什么挂等等。目前比较流行的测试框架有：
- [Jasmine](https://jasmine.github.io/)： 自带断言（assert）,mock功能
- [Mocha](https://mochajs.org/)： 框架不带断言和mock功能，需要结合其他工具，由tj大神开发
- [Jest](https://facebook.github.io/jest/)： 由Facebook出品的测试框架，在Jasmine测试框架上演变开发而来

### 断言库
断言库提供了很多语义化的方法来对值做各种各样的判断。
- [chai](http://chaijs.com/): 目前比较流行的断言库，支持 TDD（assert），BDD（expect、should）两种风格
- [should.js](https://shouldjs.github.io/)：也是tj大神所写

### mock库
- [sinon.js](http://sinonjs.org/)：使用Sinon，我们可以把任何JavaScript函数替换成一个测试替身。通过配置，测试替身可以完成各种各样的任务来让测试复杂代码变得简单。支持 spies, stub, fake XMLHttpRequest, Fake server, Fake time，很强大


### 测试集成管理工具
- [karma](https://karma-runner.github.io/2.0/index.html)：Google Angular 团队写的，功能很强大，有很多插件。可以连接真实的浏览器跑测试。能够用一些测试覆盖率统计的工具统计一下覆盖率；或是能够加入持续集成，提交代码后自动跑测试用例。


## 测试脚本的写法
通常，测试脚本与所要测试的源码脚本同名，但是后缀名为.test.js（表示测试）或者.spec.js（表示规格）。

```
// add.test.js
var add = require('./add.js');
var expect = require('chai').expect;

describe('加法函数的测试', function() {
  it('1 加 1 应该等于 2', function() {
    expect(add(1, 1)).to.be.equal(2);
  });
});
```
上面这段代码，就是测试脚本，它可以独立执行。测试脚本里面应该包括一个或多个describe块，每个describe块应该包括一个或多个it块。

describe块称为"测试套件"（test suite），表示一组相关的测试。它是一个函数，第一个参数是测试套件的名称（"加法函数的测试"），第二个参数是一个实际执行的函数。

describe干的事情就是给测试用例分组。为了尽可能多的覆盖各种情况，测试用例往往会有很多。这时候通过分组就可以比较方便的管理（这里提一句，describe是可以嵌套的，也就是说外层分组了之后，内部还可以分子组）。另外还有一个非常重要的特性，就是每个分组都可以进行预处理（before、beforeEach）和后处理（after, afterEach）。

it块称为"测试用例"（test case），表示一个单独的测试，是测试的最小单位。它也是一个函数，第一个参数是测试用例的名称（"1 加 1 应该等于 2"），第二个参数是一个实际执行的函数。

大型项目有很多测试用例。有时，我们希望只运行其中的几个，这时可以用only方法。describe块和it块都允许调用only方法，表示只运行某个测试套件或测试用例。此外，还有skip方法，表示跳过指定的测试套件或测试用例。
```
describe.only('something', function() {
  // 只会跑包在里面的测试
})

it.only('do do', () => {
  // 只会跑这一个测试
})
```

## react 单测示例一
![](mocha.jpg)
[react 单元测试框架 demo1](https://github.com/CharmSun/react-unit-test-mocha-demo)

该框架采用 karma + mocha + chai + sinon 的组合， 是一种采用工具较多，同时自由度较高的解决方案。虽然工具库使用的较多，但有助于理解各个工具库的作用和使用，也有助于加深对前端单元测试的理解。

其中React的测试库使用 enzyme，React测试必须使用官方的测试工具库，但是它用起来不够方便，所以有人做了封装，推出了一些第三方库，其中Airbnb公司的Enzyme最容易上手。

关于该库的 api 使用可参考：

[官方文档](http://airbnb.io/enzyme/)

[阮老师React测试入门](http://www.ruanyifeng.com/blog/2016/02/react-testing-tutorial.html)

## react 单测示例二
![](jest.png)
[react 单元测试框架 demo2](https://github.com/CharmSun/react-unit-test-jest-demo)

该框架只采用了Jest，是比较简洁的方案，同样也使用了 enzyme。

[Jest](https://facebook.github.io/jest/) 是Facebook开发的一个测试框架，它集成了测试执行器、断言库、spy、mock、snapshot和测试覆盖率报告等功能。React项目本身也是使用Jest进行单测的，因此它们俩的契合度相当高。
之前仅在其内部使用，后开源，并且是在Jasmine测试框架上演变开发而来，使用了熟知的expect(value).toBe(other)这种断言格式。

PS: 目前 enzyme 使用时需要加入设置如下：
```
import Enzyme from 'enzyme';
import Adapter from 'enzyme-adapter-react-16';

Enzyme.configure({ adapter: new Adapter() });
```
上面两个框架方案中都有加入该配置的方法，详见示例。



## 参考
- [聊一聊前端自动化测试](https://www.cnblogs.com/haochuang/articles/7941213.html)
- [前端自动化单元测试初探](https://www.jianshu.com/p/6726c0410650)
- [Javascript的Unit Test](http://www.tych.io/tech/2013/07/10/unit-test.html)
- [单元测试all in one](http://react-china.org/t/all-in-one/15990)
- [Sinon指南: 使用Mocks, Spies 和 Stubs编写JavaScript测试](http://www.zcfy.cc/article/sinon-tutorial-javascript-testing-with-mocks-spies-stubs-422.html#)
- [测试框架 Mocha 实例教程](http://www.ruanyifeng.com/blog/2015/12/a-mocha-tutorial-of-examples.html)

