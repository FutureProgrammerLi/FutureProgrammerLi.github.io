---
title: 类组件和hooks函数式组件有什么区别?
date: 2020-10-01 11:24:39
tags: class & hooks
categories:
- React
- Blog
---

> 原文地址：https://overreacted.io/how-are-function-components-different-from-classes/
>
> 作者： Dan Abramov
>
> 译者: Li
>
> ps: 个人觉得翻译能够更好的记住看过的东西,所以,尝试翻译一下啦.虽然英语蹩脚,还是希望能一点点进步啦~(英语或者编程方面)

# 类组件和函数组件有什么不同?

React里,函数组件和类组件有什么不同呢?

过去一段时间内,这个问题的经典答案是: 类组件能够提供更多的特性(比如类).然而,Hooks的诞生使得这个答案变得不再正确.

可能你听说过,类组件比函数组件有着更好的性能.性能的哪一方面呢?大多性能指标其实还是不完整的,所以我就不妄下定论了.性能的好坏其实是由代码做了什么,而不是说用类组件还是函数组件来实现来决定的.据我们的观察,两种组件的性能差别微乎其微,尽管它们背后采取的优化策略不尽相同.

除非你有其它的原因或者不介意当hooks的'小白鼠'(early adopter),不然我们并不建议重写已有的类组件.Hooks还是新的技术(就像React在2014年还是一个新框架一样),一些最佳实践并没能得到足够的发掘,出现在一些教程当中.

所以说了这么多,Hooks到底给我们带来了什么?Hooks和Classes到底是有什么底层性的差异吗?当然有--心智模型(mental model)上有根本的不同.在这篇文章中,我将展开说说它们最大的区别.它们的差异从2015年函数式组件这一概念被推出时就一直存在,但它经常会被忽略

>  **Function components capture the rendered values**

函数组件会捕获渲染时的所有值(水平有限,根据个人认识自由扩展..)

我们就展开说说,这句话到底什么意思.

---

注意: 这篇文章并不是在判断类组件和函数组件到底哪个更好.我只是在介绍这两种React的编程模式的差异.如果你在使用函数组件时遇到问题,去看看<a href='https://reactjs.org/docs/hooks-faq.html#adoption-strategy'>Hooks FAQ</a>

---

想一想以下这一组件:

```js
function ProfilePage(props) {
  const showMessage = () => {
    alert('Followed ' + props.user);
  };

  const handleClick = () => {
    setTimeout(showMessage, 3000);
  };

  return (
    <button onClick={handleClick}>Follow</button>
  );
}
```

这个组件会展示一个按钮.为了模拟网络请求,点击按钮3秒后会出现一个确认框.比如,如果props.user的值为'Dan',那么点击按钮三秒后会出现'Followed Dan' 的确认框.够简单吧?

(这里无论用箭头函数还是声明式函数结果都是一样的)

如果用类写以上组件,我们会怎么做呢?

```js
class ProfilePage extends React.Component {
  showMessage = () => {
    alert('Followed ' + this.props.user);
  };

  handleClick = () => {
    setTimeout(this.showMessage, 3000);
  };

  render() {
    return <button onClick={this.handleClick}>Follow</button>;
  }
}
```

我们多数人都会认为,以上两个代码段是等价的.所以我们总是在这两种模式上随意切换,而没有注意到一些指示(implications)

**其实,这两段代码还是有一些细微的不同的**,好好看一下这两段代码,你能找到它们的差异吗?我自己(作者)也是看了好久才能看出它们的不同.

接下来我们将解释它们的不同,而且为什么这些不同很是重要.如果你想自己找出它们的不同,试着从这个<a href='https://codesandbox.io/s/pjqnl16lm7'>demo</a>里找吧.

---

我还是想强调一下,我这里说的差异本质上跟Hooks其实没有直接联系.上面的例子就没有用hooks对吧?

这里的差异其实是函数和类在React中的不同.如果你想在React应用中更多的使用函数,那么这些差异你还是需要理解的.

---

**我们将通过一个在React应用中经常出现的一个bug,来讲述类和函数在React中的差异.**

打开一下这个<a href='https://codesandbox.io/s/pjqnl16lm7'>demo</a>,其实只是上面两个代码段加上一些展示,然后按以下顺序分别操作一下两个按钮.

1. 点击一下Follow按钮(类或函数任意一个)
2. 点击之后的三秒内切换到另一个主页(profile)
3. 看看alert里面,你follow了谁?

你会看到一个奇怪的差异

* 如果你点了函数式组件那个按钮,按下按钮时选的是Dan的主页.然后你切换到Sophie的主页.三秒过后,alert里的文字依然是提示你关注了Dan的主页.
* 如果你点的是类组件.alert会提示你关注了Sophie的主页.

![Demonstration of the steps](https://overreacted.io/386a449110202d5140d67336a0ade5a0/bug.gif)

显然在这个例子中,函数组件表现的行为才是正确的.**如果我关注了一个人然后切换到别的主页,组件不应该纠结我到底关注了谁**(正常思维当然是先关注后切换,而不是关注了切换之后的那个,译者自己的理解).

(不过你关注了<a href='https://mobile.twitter.com/sophiebits'>Sophie</a>也不亏) (233333)

---

所以为什么类组件会这样表现呢?

我们仔细看看类组件中的showMessage这个方法:

```js
class ProfilePage extends React.Component {
  showMessage = () => {
    alert('Followed ' + this.props.user);
  };
```

这个类方法依赖属性this.props.user.Props在React中是不可改变的.**但是,`this`可以,而且它总有可能会被改变**

实话说,`this`在类组件中可以改变,也是它的一大特点.React本身会改变`this`指向(?),从而让你能够在render和生命周期函数内读到最新的`this`指向.

所以如果我们的组件在请求的时候(前面模拟的3s)重新渲染了,this.props就会被改变,而showMessage这个方法也总会读到最新的props值.

以上的用户操作实际上暴露一个有趣的点:如果说UI在概念上是一个函数执行当前状态下的结果,**那么事件处理器就是渲染结果的一部分--就像渲染UI一样**.事件处理器应该属于某一次特定的渲染,有着属于那一次渲染时的props和state.

然而,用回调函数中,依赖this.props的定时任务会打破这样的联系.`showMessage`这一回调并非属于某次特定的渲染,所以它并没读到正确的props.它是通过`this`来读取props的.(能懂吗?)

---

**如果没有函数组件,我们要怎么解决这个问题呢?**

我们要'修复'render和this的联系,让render读到某个特定时刻的this.props. `props`好像在执行的时候迷失了(断掉了某种联系).

解决方法之一是,提前读取`this.props`并将user提取出来传到`showMessage`里面.

```js
class ProfilePage extends React.Component {
  showMessage = (user) => {
    alert('Followed ' + user);
  };

  handleClick = () => {
    const {user} = this.props;//利用闭包cover当前的user?
    setTimeout(() => this.showMessage(user), 3000);
  };

  render() {
    return <button onClick={this.handleClick}>Follow</button>;
  }
```

这确实可行.但是扩展组件时很可能会变得复杂,容易出错.如果我们依赖的不仅仅是一个props呢?如果我们要访问到state呢?**如果`showMessage`调用了另一个方法,那一个方法又需要从props或者state中的值,我们还是会遇到同样的问题.所以我们不得不把整个state(this,state)或者整个props(this.props)以参数形式传给`showMessage`里调用的其它方法.(好麻烦)

如果真这样做了,类组件基本就没执行效率可言了.而且还很容易出bug.

相似地,直接把showMessage以行内形式直接嵌在handleClick里面也还是会遇到同样问题.我们要把定时任务里的回调写在外面,还要读到那一次渲染的,正确的props或state.**这一问题不只是用React时会遇到--不少UI库,把数据放在一个可变的对象(如`this`)中的,都可以重现这个问题.**

或许,我们可以在constructor中绑定这个方法?

```js
class ProfilePage extends React.Component {
  constructor(props) {
    super(props);
    this.showMessage = this.showMessage.bind(this);
    this.handleClick = this.handleClick.bind(this);
  }

  showMessage() {
    alert('Followed ' + this.props.user);
  }

  handleClick() {
    setTimeout(this.showMessage, 3000);
  }

  render() {
    return <button onClick={this.handleClick}>Follow</button>;
  }
}
```

并不行.(!!!???那你还???)我们是读props读得太晚了,而不是使用的语法错了.**不过,我们能够依靠JavaScript的闭包特性解决这一问题**

我们尽量避免使用闭包,因为很难确定被闭包包裹(close over/covered)的值是否能被改变.但在React中,props和state都是不可改变的(或者,它不应该被改变). 这不就是我们想要的吗?(removes the major footgun of closure).

就是说如果你把某次渲染的props或者state用闭包维持下来,那么你总能依赖它们,它们总是不变的.

```js
class ProfilePage extends React.Component {
  render() {
    // Capture the props!
    const props = this.props;

    // Note: we are *inside render*.
    // These aren't class methods.
    const showMessage = () => {
      alert('Followed ' + props.user);
    };

    const handleClick = () => {
      setTimeout(showMessage, 3000);
    };

    return <button onClick={handleClick}>Follow</button>;
  }
}// 好家伙,直接把所有方法写render里面.外面还有什么用!?
```

**这样你就把渲染时的所有props捕获起来啦!**

![Capturing Pokemon](https://overreacted.io/fa483dd5699aac1350c57591770a49be/pokemon.gif)

这样render里的所有代码(包括`showMessage`)都能确保是属于某次特定的渲染了.React就不会'动我们的奶酪'啦!(move our cheese)

**之后就可以把我们的辅助函数都添加到这里面,而且它们使用的都是正确的props和state了**.闭包救我狗命呐!(Closures to the rescue!)

---

<a href='https://codesandbox.io/s/oqxy9m7om5'>上面例子</a>的表现行为就是我们想要的,但它看起来有点怪.把所有的函数都放在`render`里面了,那还用class干嘛?

那确实,不如我们直接把class给移除掉吧?

```js
function ProfilePage(props) {
  const showMessage = () => {
    alert('Followed ' + props.user);
  };

  const handleClick = () => {
    setTimeout(showMessage, 3000);
  };

  return (
    <button onClick={handleClick}>Follow</button>
  );
}
```

以上的做法会捕获(capture)`props` --React以参数的形式传入给组件.**不过不像`this`的行为,React并不会改变props这个对象**.

如果你直接在函数定义时直接把props解构,这样看起来会更加的明显.

```js
function ProfilePage({ user }) {
  const showMessage = () => {
    alert('Followed ' + user);
  };

  const handleClick = () => {
    setTimeout(showMessage, 3000);
  };

  return (
    <button onClick={handleClick}>Follow</button>
  );
}
```

当父组件传不同的props给子组件ProfilePage时,React会再次调用ProfilePage这个函数.不过,我们点击时的事件处理器是属于上一次调用的,它有着属于自己的user值,showMessage也会读取这个属于那一次渲染的user值.它们内部其实什么都没改变.

这就是为什么在这个<a href='https://codesandbox.io/s/pjqnl16lm7'>demo</a>中,先关注Sophie,切换到Sunil主页,alert会提示'Followed Sophie'.

![Demo of correct behavior](https://overreacted.io/84396c4b3982827bead96912a947904e/fix.gif)这样的行为是正确的(尽管你想两个都关注, <a href='https://mobile.twitter.com/threepointone'>Sunil's twitter</a>

---

所以,读到这里,我们应该能够明白在React中函数和类的区别:

>**Function components capture the rendered values**.
>
>函数组件捕获渲染时的所有值.

**有了Hooks, state也是要遵从这一原则的**(什么原则?).看看这个例子:

```js
function MessageThread() {
  const [message, setMessage] = useState('');

  const showMessage = () => {
    alert('You said: ' + message);
  };

  const handleSendClick = () => {
    setTimeout(showMessage, 3000);
  };

  const handleMessageChange = (e) => {
    setMessage(e.target.value);
  };

  return (
    <>
      <input value={message} onChange={handleMessageChange} />
      <button onClick={handleSendClick}>Send</button>
    </>
  );
}
```

这是<a href='https://codesandbox.io/s/93m5mz9w24'>demo</a>地址.

尽管这demo并没有好的UI提示,但它阐述了同一个点:如果我要发送一条信息,组件不应该对我发送的是哪条信息而疑惑甚至出错.这个函数组件返回捕获了当时state的回调函数,让浏览器调用.所以当我按下Send时,`message`的值总是我输入框内的值.

---

这样我们就知道了,React里的函数默认就会捕获props和state.**但是,如果我们想读到最新的props或者state而不是属于某次特定渲染时的呢?**(用function components实现class components的行为) 如果我们想要'读取未来的props和state'呢?

在类组件中,你直接用this.props或者this.state就可以了,因为`this`指向是会被React改变的.而在函数组件中,你也可以用一个可变的值来贯穿组件的所有renders.这个值叫'ref'.

(个人理解,类组件的this,函数组件中的ref,拿到的总是最新的.)

```js
function MyComponent() {
  const ref = useRef(null);
  // You can read or write `ref.current`.
  // ...
}
```

但是,你必须自己管理好ref(React会内部管理this).

ref的作用就跟实例的this作用一样.它就是在滚滚泥石流中的清流(immutable in the mutable imperative world),处处讲究不变的世界中,时刻保持变化的值.你可能熟悉DOM的refs,但这里的ref意义更加广泛.这就是一个盒子,你想摆什么进去就摆什么.

甚至直观上讲, this.something和something.current是一样的.它们所代表的意义也是一样的.

在函数组件中,React默认并不会创建任何指向props或state的ref(this.props/this.state).大多数情况下你并不需要这些refs,就算创建了也是浪费.然而你可以手动的跟踪这些值的变化,if you like:

```js
function MessageThread() {
  const [message, setMessage] = useState('');
  const latestMessage = useRef('');

  const showMessage = () => {
    alert('You said: ' + latestMessage.current);
  };

  const handleSendClick = () => {
    setTimeout(showMessage, 3000);
  };

  const handleMessageChange = (e) => {
    setMessage(e.target.value);
    latestMessage.current = e.target.value;
  };
```

如果在`showMessage`中读`message`的值,我们可以看到按下Send时的值.但当我们读取`latestMessage.current`时,我们却能读到最新的值--尽管我们按了Send按钮.(不用current,按下Send时input的值是什么就是什么.用current,按下Send后继续输入,alert里的值会出现你继续输入的那些内容.就是那3秒内操作的区别)

还是不懂的话,看下这<a href='https://codesandbox.io/s/93m5mz9w24'>两个</a><a href='https://codesandbox.io/s/ox200vw8k9'>|demo</a>吧.ref就是万物皆不变,唯它变的选择,而且它在一些场合还是很方便的.

一般而言,你应该避免在渲染的时候读取或设置refs,因为它们都是可变的.我们想要的是'可预测的渲染'.**然而如果我们想要拿到某个props或state最新的值,我们就要手动的更新ref的指向,那就很烦了**,不过,我们可以结合useEffect来自动改变ref的指向:

```jsx
function MessageThread() {
  const [message, setMessage] = useState('');

  // Keep track of the latest value.  
  const latestMessage = useRef('');  
  useEffect(() => {    
      latestMessage.current = message;  
  });
  const showMessage = () => {
  alert('You said: ' + latestMessage.current);  
  };
}
```

看看这个<a href='https://codesandbox.io/s/yqmnz7xy8x'>demo</a>.

我们在effect里面将message赋值给current,这样ref的值就会只在DOM被更新之后才会改变.这样我们就不会破坏像官方例子<a href='https://reactjs.org/blog/2018/03/01/sneak-peek-beyond-react-16.html'>Time Slicing and Suspense</a>这样依赖可变渲染的特性了(interruptable rendering).

这样在effect里改变ref的使用场景其实并不多.**捕获props或state才是更好的默认行为**,不过,当你用到一些特定API,如intervals和subscriptions时这种方式还是很方便的.因为你可以用ref来跟踪任何的值--prop,state的某个值,或者整个props对象,甚至是一个函数.

这种模式对于优化而言也会很方便--比如`useCallback`过于频繁改变时.不过,使用reducer更好.

---

在这篇文章中,我们探讨了一些类组件常见,却糟透了的(broken)模式,以及如何使用闭包来解决问题.不过,你可能会遇到过这样的情景:当你尝试在hooks中通过第二个参数列出函数所依赖的值时,会因'过气闭包'(stale closures)而遇到一些bug.这意味着使用闭包是问题吗?我并不这样认为.

正如我们上面所见,闭包帮我们解决了一些不易被发现的小问题.而且,它让我们更容易写出并行模式下运行正确的代码.或许是因为闭包在渲染时把正确的props和state给包裹了起来.

据我目前看到所有的案例中,**'过气闭包'的问题是由于人们错误的认为'函数是不会变的'或者'props总是相同的'而造成的**,这不是重点,我只是想在这篇文章中简单澄清一下(clarify).

函数包裹了它们自己的props和state -- 所以它们'身份'(identity)是很重要的. 这并不是bug,而是函数组件的一个特性.比如,不应该在useEffect或useCallback中只定义函数,而不列出函数的依赖项(不使用第二个参数)(正确方式应该是使用useReducer或者useRef -- 我们将很快在官方文档中介绍如何选择这两种方法).

当我们写React时用到大量的函数时,我们需要知道如何优化代码,以及清楚哪些值会在执行期间被改变.

就像Fredrik说的那样:

> **The best mental rule I've found so far with hooks is "code as if any value can change at any time"**
>
> 我发现用hooks最好的心智模式是认为所有的值在任何时候都可能会被改变.

函数当然也遵从这样的心智模式.在各种React教程中贯彻这种理念还是要花点时间的.它需要在原有的,类组件的思维方式上作出调整.不过我希望这篇文章能够帮你实现一小点这种转变.

React functions always capture their values -- and now we know why.

![Smiling Pikachu](https://overreacted.io/fc3bddf6d4ca14bc77917ac0cfad3608/pikachu.gif)

They're a whole different Pokémon.