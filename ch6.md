# JavaScript 轻量级函数式编程
# 第 6 章：值的不可变性

在第 5 章中，我们探讨了减少副作用的重要性：副作用是引起程序意外状态改变的原因，同时也可能会带来意想不到的惊喜（bugs）。这样的暗雷在程序中出现的越少，开发者对程序的信心无疑就会越强，同时代码的可读性也会越高。本章的主题，将继续朝减少程序副作用的方向努力。

如果编程风格幂等性是指定义一个数据变更操作以便只影响一次程序状态，那么现在我们将注意力转向将这个影响次数从 1 降为 0。

现在我们开始探索值的不可变性，即只在我们的程序中使用不可被改变的数据。

## 原始值的不可变性

原始数据类型（`number`、`string`、`boolean`、`null` 和 `undefined`）本身就是不可变的；无论如何你都没办法改变它们。

```js
// 无效，且毫无意义
2 = 2.5;
```

然而 JS 确实有一个特性，使得看起来允许我们改变原始数据类型的值, 即“boxing”特性。当你访问原始类型数据时 —— 特别是 `number`、`string` 和 `boolean` —— 在这种情况下，JS 会自动的把它们包裹（或者说“包装”）成这个值对应的对象（分别是 `Number`、`String` 以及 `Boolean`）。

思考下面的代码：

```js
var x = 2;

x.length = 4;

x;				// 2
x.length;		// undefined
```

数值本身并没有可用的 `length` 属性，因此 `x.length = 4` 这个赋值操作正试图添加一个新的属性，不过它静默地失败了（也可以说是这个操作被忽略了或被抛弃了，这取决于你怎么看）；变量 `x` 继续承载那个简单的原始类型数据 —— 数值 `2`。

但是 JS 允许 `x.length = 4` 这条语句正常执行的事实着实令人困惑。如果这种现象真的无缘无故出现，那么代码的阅读者无疑会摸不着头脑。好消息是，如果你使用了严格模式（`"use strict";`），那么这条语句就会抛出异常了。

那么如果尝试改变那些明确被包装成对象的值呢？

```js
var x = new Number( 2 );

// 没问题
x.length = 4;
```

这段代码中的 `x` 保存了一个对象的引用，因此可以正常地添加或修改自定义属性。

像 `number` 这样的原始数型，值的不可变性看起来相当明显，但字符串呢？JS 开发者有个共同的误解 —— 字符串和数组很像，所以应该是可变的。JS 使用 `[]` 访问字符串成员的语法甚至还暗示字符串真的就像数组。不过，字符串的确是不可变的：

```js
var s = "hello";

s[1];				// "e"

s[1] = "E";
s.length = 10;

s;					// "hello"
```

尽管可以使用 `s[1]` 来像访问数组元素一样访问字符串成员，JS 字符串也并不是真的数组。`s[1] = "E"` 和 `s.length = 10` 这两个赋值操作都是失败的，就像刚刚的 `x.length = 4` 一样。在严格模式下，这些赋值都会抛出异常，因为 `1` 和 `length` 这两个属性在原始数据类型字符串中都是只读的。

有趣的是，即便是包装后的 `String` 对象，其值也会（在大部分情况下）表现的和非包装字符串一样 —— 在严格模式下如果改变已存在的属性，就会抛出异常：

```js
"use strict";

var s = new String( "hello" );

s[1] = "E";			// error
s.length = 10;		// error

s[42] = "?";		// OK

s;					// "hello"
```

## 从值到值

我们将在本节详细展开从值到值这个概念。但在开始之前应该心中有数：值的不可变性并不是说我们不能在程序编写时不改变某个值。如果一个程序的内部状态从始至终都保持不变，那么这个程序肯定相当无趣！它同样不是指变量不能承载不同的值。这些都是对值的不可变这个概念的误解。

值的不可变性是指当需要改变程序中的状态时，我们不能改变已存在的数据，而是必须创建和跟踪一个新的数据。

例如：

```js
function addValue(arr) {
	var newArr = [ ...arr, 4 ];
	return newArr;
}

addValue( [1,2,3] );	// [1,2,3,4]
```

注意我们没有改变数组 `arr` 的引用，而是创建了一个新的数组（`newArr`），这个新数组包含数组 `arr` 中已存在的值，并且新增了一个新值 `4`。

使用我们在第 5 章讨论的副作用的相关概念来分析 `addValue(..)`。它是纯的吗？它是否具有引用透明性？给定相同的数组作为输入，它会永远返回相同的输出吗？它无副作用吗？**答案是肯定的。**

设想这个数组 `[1, 2, 3]`, 它是由先前的操作产生，并被我们保存在一个变量中，它代表着程序当前的状态。我们想要计算出程序的下一个状态，因此调用了 `addValue(..)`。但是我们希望下一个状态计算的行为是直接的和明确的，所以 `addValue(..)` 操作简单的接收一个直接输入，返回一个直接输出，并通过不改变 `arr` 引用的原始数组来避免副作用。

这就意味着我们既可以计算出新状态 `[1, 2, 3, 4]`，也可以掌控程序的状态变换。程序不会出现过早的过渡到这个状态或完全转变到另一个状态（如 `[1, 2, 3, 5]`）这样的意外情况。通过规范我们的值并把它视为不可变的，我们大幅减少了程序错误，使我们的程序更易于阅读和推导，最终使程序更加可信赖。

`arr` 所引用的数组是可变的，只是我们选择不去改变他，我们实践了值不可变的这一精神。

同样的，可以将“以拷贝代替改变”这样的策略应用于对象，思考下面的代码：

```js
function updateLastLogin(user) {
	var newUserRecord = Object.assign( {}, user );
	newUserRecord.lastLogin = Date.now();
	return newUserRecord;
}

var user = {
	// ..
};

user = updateLastLogin( user );
```

### 消除本地影响

下面的代码能够体现不可变性的重要性：

```js
var arr = [1,2,3];

foo( arr );

console.log( arr[0] );
```

从表面上讲，你可能认为 `arr[0]` 的值仍然为 `1`。但事实是否如此不得而知，因为 `foo(..)` 可能会改变你传入其中的 `arr` 所引用的数组。

在之前的章节中，我们已经见到过用下面这种带有欺骗性质的方法来避免意外：

```js
var arr = [1,2,3];

foo( arr.slice() );			// 哈！一个数组副本！

console.log( arr[0] );		// 1
```

当然，使得这个断言成立的前提是 `foo` 函数不会忽略我们传入的参数而直接通过相同的 `arr` 这个自由变量词法引用来访问源数组。

对于防止数据变化负面影响，稍后我们会讨论另一种策略。

## 重新赋值

在进入下一个段落之前先思考一个问题 —— 你如何描述“常量”？

…

你可能会脱口而出“一个不能改变的值就是常量”，“一个不能被改变的变量”等等。这些回答都只能说接近正确答案，但却并不是正确答案。对于常量，我们可以给出一个简洁的定义：一个无法进行重新赋值（reassignment）的变量。

我们刚刚在“常量”概念上的吹毛求疵其实是很有必要的，因为它澄清了常量与值无关的事实。无论常量承载何值，该变量都不能使用其他的值被进行重新赋值。但它与值的本质无关。

思考下面的代码：

```js
var x = 2;
```

我们刚刚讨论过，数据 `2` 是一个不可变的原始值。如果将上面的代码改为：

```js
const x = 2;
```

`const` 关键字的出现，作为“常量声明”被大家熟知，事实上根本没有改变 `2` 的本质，因为它本身就已经不可改变了。

下面这行代码会抛出错误，这无可厚非：

```js
// 尝试改变 x，祝我好运！
x = 3;		// 抛出错误！
```

但再次重申，我们并不是要改变这个数据，而是要对变量 `x` 进行重新赋值。数据被卷进来纯属偶然。

为了证明 `const` 和值的本质无关，思考下面的代码：

```js
const x = [ 2 ];
```

这个数组是一个常量吗？**并不是。** `x` 是一个常量，因为它无法被重新赋值。但下面的操作是完全可行的：

```js
x[0] = 3;
```

为何？因为尽管 `x` 是一个常量，数组却是可变的。

关于 `const` 关键字和“常量”只涉及赋值而不涉及数据语义的特性是个又臭又长的故事。几乎所有语言的高级开发者都踩 `const` 地雷。事实上，Java 最终不赞成使用 `const` 并引入了一个全新的关键词 `final` 来区分“常量”这个语义。

抛开混乱之后开始思考，如果 `const` 并不能创建一个不可变的值，那么它对于函数式编程者来说又还有什么重要的呢？

### 意图

`const` 关键字可以用来告知阅读你代码的读者该变量不会被重新赋值。作为一个表达意图的标识，`const` 被加入 JavaScript 不仅常常受到称赞，也普遍提高了代码可读性。

在我看来，这是夸大其词，这些说法并没有太大的实际意义。我只看到了使用这种方法来表明意图的微薄好处。如果使用这种方法来声明值的不可变性，与已使用几十年的传统方式相比，`const` 简直太弱了。

为了证明我的说法，让我们来做一个实践。`const` 创建了一个在块级作用域内的变量，这意味着该变量只能在其所在的代码块中被访问：

```js
// 大量代码

{
	const x = 2;

	// 少数几行代码
}

// 大量代码
```

通常来说，代码块的最佳实践是用于仅包裹少数几行代码的场景。如果你有一个包含了超过 10 行的代码块，那么大多数开发者会建议你重构这一段代码。因此 `const x = 2` 只作用于下面的9行代码。

程序的其他部分不会影响 `x` 的赋值。

我要说的是：上述程序的可读性与下面这样基本相同：

```js
// 大量代码

{
	let x = 2;

	// 少数几行代码
}

// 大量代码
```

其实只要查看一下在 `let x = 2`; 之后的几行代码，就可以判断出 x 这个变量是否被重新赋值过了。对我来说，“实际上不进行重新赋值”相对“使用容易迷惑人的 `const` 关键字告诉读者‘不要重新赋值’”**是一个更明确的信号**。

此外，让我们思考一下，乍看这段代码起来可能给读者传达什么：

```js
const magicNums = [1,2,3,4];

// ..
```

读者可能会（错误地）认为，这里使用 `const` 的用意是你永远不会修改这个数组 —— 这样的推断对我来说合情合理。想象一下，如果你的确允许 `magicNums` 这个变量所引用的数组被修改，那么这个 `const` 关键词就极具混淆性了 —— 的很确容易发生意外，不是吗？

更糟糕的是，如果你在某处故意修改了 `magicNums`，但对读者而言不够明显呢？读者会在后面的代码里（再次错误地）认为 `magicNums` 的值仍然是 `[1, 2, 3, 4]`。因为他们猜测你之前使用 `const` 的目的就是“这个变量不会改变”。

我认为你应该使用 `var` 或 `let` 来声明那些你会去改变的变量，它们确实相比 `const` 来说**是一个更明确的信号**。

`const` 所带来的问题还没讲完。还记得我们在本章开头所说的吗？值的不可变性是指当需要改变某个数据时，我们不应该直接改变它，而是应该使用一个全新的数据。那么当新数组创建出来后，你会怎么处理它？如果你使用 `const` 声明变量来保存引用吗，这个变量的确没法被重新赋值了，那么……然后呢？

从这方面来讲，我认为 `const` 反而增加了函数式编程的困难度。我的结论是：`const` 并不是那么有用。它不仅造成了不必要的混乱，也以一种很不方便的形式限制了我们。我只用 `const` 来声明简单的常量，例如：

```js
const PI = 3.141592;
```

`3.141592` 这个值本身就已经是不可变的，并且我也清楚地表示说“`PI` 标识符将始终被用于代表这个字面量的占位符”。对我来说，这才是 `const` 所擅长的。坦白讲，我在编码时并不会使用很多这样的声明。

我写过很多，也阅读过很多 JavaScript 代码，我认为由于重新赋值导致大量的 bug 这只是个想象中的问题，实际并不存在。

我们应该担心的，并不是变量是否被重新赋值，而是**值是否会发生改变**。为什么？因为值是可被携带的，但词法赋值并不是。你可以向函数中传入一个数组，这个数组可能会在你没意识到的情况下被改变。但是你的其他代码在预期之外重新给变量赋值，这是不可能发生的。

### 冻结

这是一种简单廉价的（勉强）将像对象、数组、函数这样的可变的数据转为“不可变数据”的方式：

```js
var x = Object.freeze( [2] );
```

`Object.freeze(..)` 方法遍历对象或数组的每个属性和索引，将它们设置为只读以使之不会被重新赋值，事实上这和使用 `const` 声明属性相差无几。`Object.freeze(..)` 也会将属性标记为“不可配置（non-reconfigurable）”，并且使对象或数组本身不可扩展（即不会被添加新属性）。实际上，而就可以将对象的顶层设为不可变。

注意，仅仅是顶层不可变！

```js
var x = Object.freeze( [ 2, 3, [4, 5] ] );

// 不允许改变：
x[0] = 42;

// oops，仍然允许改变：
x[2][0] = 42;
```

`Object.freeze(..)` 提供浅层的、初级的不可变性约束。如果你希望更深层的不可变约束，那么你就得手动遍历整个对象或数组结构来为所有后代成员应用 `Object.freeze(..)`。

与 `const` 相反，`Object.freeze(..)` 并不会误导你，让你得到一个“你以为”不可变的值，而是真真确确给了你一个不可变的值。

回顾刚刚的例子：

```js
var arr = Object.freeze( [1,2,3] );

foo( arr );

console.log( arr[0] );			// 1
```

可以非常确定 `arr[0]` 就是 `1`。

这是非常重要的，因为这可以使我们更容易的理解代码，当我们将值传递到我们看不到或者不能控制的地方，我们依然能够相信这个值不会改变。

## 性能

每当我们开始创建一个新值（数组、对象等）取代修改已经存在的值时，很明显迎面而来的问题就是：这对性能有什么影响？

如果每次想要往数组中添加内容时，我们都必须创建一个全新的数组，这不仅占用 CPU 时间并且消耗额外的内存。不再存在任何引用的旧数据将会被垃圾回收机制回收；更多的 CPU 资源消耗。

这样的取舍能接受吗？视情况而定。对代码性能的优化和讨论**都应该有个上下文**。

如果在你的程序中，只会发生一次或几次单一的状态变化，那么扔掉一个旧对象或旧数组完全没必要担心。性能损失会非常非常小 —— 顶多只有几微秒 —— 对你的应用程序影响甚小。追踪和修复由于数据改变引起的 bug 可能会花费你几分钟甚至几小时的时间，这么看来那几微秒简直没有可比性。

但是，如果频繁的进行这样的操作，或者这样的操作出现在应用程序的核心逻辑中，那么性能问题 —— 即性能和内存 —— 就有必要仔细考虑一下了。

以数组这样一个特定的数据结构来说，我们想要在每次操作这个数组时使每个更改都隐式地进行，就像结果是一个新数组一样，但除了每次都真的创建一个数组之外，还有什么其他办法来完成这个任务呢？像数组这样的数据结构，我们期望除了能够保存其最原始的数据，然后能追踪其每次改变并根据之前的版本创建一个分支。

在内部，它可能就像一个对象引用的链表树，树中的每个节点都表示原始值的改变。从概念上来说，这和 **git** 的版本控制原理类似。


<p align="center">
	<img src="fig18.png" width="490">
</p>

想象一下使用这个假设的、专门处理数组的数据结构：

```js
var state = specialArray( 1, 2, 3, 4 );

var newState = state.set( 42, "meaning of life" );

state === newState;					// false

state.get( 2 );						// 3
state.get( 42 );					// undefined

newState.get( 2 );					// 3
newState.get( 42 );					// "meaning of life"

newState.slice( 1, 3 );				// [2,3]
```

`specialArray(..)` 这个数据结构会在内部追踪每个数据更新操作（例如 `set(..)`），类似 *diff*，因此不必要为原始的那些值（`1`、`2`、`3` 和 `4`）重新分配内存，而是简单的将 `"meaning of life"` 这个值加入列表。重要的是，`state` 和 `newState` 分别指向两个“不同版本”的数组，因此**值的不变性这个语义得以保留**。

发明你自己的性能优化数据结构是个有趣的挑战。但从实用性来讲，找一个现成的库会是个更好的选择。**Immutable.js**（http://facebook.github.io/immutable-js） 是一个很棒的选择，它提供多种数据结构，包括 `List`（类似数组）和 `Map`（类似普通对象）。

思考下面的 `specialArray` 示例，这次使用 `Immutable.List`：

```js
var state = Immutable.List.of( 1, 2, 3, 4 );

var newState = state.set( 42, "meaning of life" );

state === newState;					// false

state.get( 2 );						// 3
state.get( 42 );					// undefined

newState.get( 2 );					// 3
newState.get( 42 );					// "meaning of life"

newState.toArray().slice( 1, 3 );	// [2,3]
```

像 Immutable.js 这样强大的库一般会采用非常成熟的性能优化。如果不使用库而是手动去处理那些细枝末节，开发的难度会相当大。

当改变值这样的场景出现的较少且不用太关心性能时，我推荐使用更轻量级的解决方案，例如我们之前提到过的内置的 `Object.freeze(..)`。

## 以不可变的眼光看待数据

如果我们从函数中接收了一个数据，但不确定这个数据是可变的还是不可变的，此时该怎么办？去修改它试试看吗？**不要这样做。** 就像在本章最开始的时候所讨论的，不论实际上接收到的值是否可变，我们都应以它们是不可变的来对待，以此来避免副作用并使函数保持纯度。

回顾一下之前的例子：

```js
function updateLastLogin(user) {
	var newUserRecord = Object.assign( {}, user );
	newUserRecord.lastLogin = Date.now();
	return newUserRecord;
}
```

该实现将 `user` 看做一个不应该被改变的数据来对待；`user` 是否真的不可变完全不会影响这段代码的阅读。对比一下下面的实现：

```js
function updateLastLogin(user) {
	user.lastLogin = Date.now();
	return user;
}
```

这个版本更容易实现，性能也会更好一些。但这不仅让 `updateLastLogin(..)` 变得不纯，这种方式改变的值使阅读该代码，以及使用它的地方变得更加复杂。

**应当总是将 user 看做不可变的值**，这样我们就没必要知道数据从哪里来，也没必要担心数据改变会引发潜在问题。

JavaScript 中内置的数组方法就是一些很好的例子，例如 `concat(..)` 和 `slice(..)` 等：

```js
var arr = [1,2,3,4,5];

var arr2 = arr.concat( 6 );

arr;					// [1,2,3,4,5]
arr2;					// [1,2,3,4,5,6]

var arr3 = arr2.slice( 1 );

arr2;					// [1,2,3,4,5,6]
arr3;					// [2,3,4,5,6]
```

其他一些将参数看做不可变数据且返回新数组的原型方法还有：`map(..)` 和 `filter(..)` 等。`reduce(..)` / `reduceRight(..)` 方法也会尽量避免改变参数，尽管它们并不默认返回新数组。

不幸的是，由于历史问题，也有一部分不纯的数组原型方法：`splice(..)`、`pop(..)`、`push(..)`、`shift(..)`、`unshift(..)`、`reverse(..)` 以及 `fill(..)`。

有些人建议禁止使用这些不纯的方法，但我不这么认为。因为一些性能面的原因，某些场景下你仍然可能会用到它们。不过你也应当注意，如果一个数组没有被本地化在当前函数的作用域内，那么不应当使用这些方法，避免它们所产生的副作用影响到代码的其他部分。

不论一个数据是否是可变的，永远将他们看做不可变。遵守这样的约定，你程序的可读性和可信赖度将会大大提升。

## 总结

值的不可变性并不是不改变值。它是指在程序状态改变时，不直接修改当前数据，而是创建并追踪一个新数据。这使得我们在读代码时更有信心，因为我们限制了状态改变的场景，状态不会在意料之外或不易观察的地方发生改变。

由于其自身的信号和意图，`const` 关键字声明的常量通常被误认为是强制规定数据不可被改变。事实上，`const` 和值的不可变性声明无关，而且使用它所带来的困惑似乎比它解决的问题还要大。另一种思路，内置的 `Object.freeze(..)` 方法提供了顶层值的不可变性设定。大多数情况下，使用它就足够了。

对于程序中性能敏感的部分，或者变化频繁发生的地方，处于对计算和存储空间的考量，每次都创建新的数据或对象（特别是在数组或对象包含很多数据时）是非常不可取的。遇到这种情况，通过类似 **Immutable.js** 的库使用不可变数据结构或许是个很棒的主意。

值不变在代码可读性上的意义，不在于不改变数据，而在于以不可变的眼光看待数据这样的约束。
