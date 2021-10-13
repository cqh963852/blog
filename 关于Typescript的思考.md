# 关于 Typescript 的思考

突然想写一篇关于 Typescript 的文章。

一是感觉做了很久的工程师，在和别人讨论时，很难拿出让人信服的观点。寄希望于用人格魅力征服别人是不可取的。所以应该写一点东西，阶段性的记录下自己的想法。

二是确实突然有了想法，好像是量到质的转变，所以记录下来。

## 来源

在写这篇文章的时候，已经累积了三年的前端工程师经验（2018-2021）。

而这次进入了一个新的团队，新的项目。虽然才只有几个月，但是有很大的感触。

## 正文

我们的新项目是一个低代码平台，我们希望用节点的形式描述页面的元素，布局，甚至是计算，流程。

开始设计类型

1. 同事希望使用 type 描述类型，而不是 interface

   我们设计了一个结构希望作为所有节点类型的基石

   ```ts
   type Node<Props extends {}> = Props && {
    id: ID;
   }
   ```

又因为使用 Recoiljs 所以我们吸收了 Recoiljs 中 Derived 的概念。

> Derived 值不能修改，由 Recoiljs 通过用户提供的 derive 函数自动计算得出。用户定义的组件使用 Derived 值。

在到这里的时候一切都还正常，因为几乎没有什么设计。但是再往后我们设计了新的结构。

1. 我们希望拓展属性是拍平一次后存在节点中。
2. 我们希望有一个结构可以描述 derive 函数中入参的节点类型。

   ```ts
   type PrimaryNode<P extends {}> = Node<P>;
   ```

3. 我们希望有一个结构可以承载 derive 函数返回值的节点类型。

   ```ts
   type DerivedNode<D extends {}> = Node<D>;
   ```

到这里的时候问题出现了。

一、当用户想拓展属性时，使用 type 拓展 object。然而这并不是一个友好的视觉体验，泛型可能会被处理，并不能体现出拓展含义。而 interface 拓展显然更合适。

```ts
type Primitives = number | string | boolean;

type CustomNode = PrimaryNode<{
  value: Primitives;
}>;

//-->

interface ICustomNode extends PrimaryNode {
  value: Primitives;
}
```

二、derive 函数的设计目标是交由用户实现的，前端组件使用 Derived 数据。DerivedNode 中的泛型以及实现方式限定了 derive 函数的使用。用户设计 derive 函数时变得非常麻烦

```ts
type Derive = <P, D>(node: PrimaryNode<P>) => DerivedNode<D>;
```

```ts
type ExtendsPrimaryNodeProps = {};

type ExtendsDerivedNodeProps = {};

const derive: Derive<ExtendsPrimaryNodeProps, ExtendsDerivedNodeProps> = (
  node
) => {};
```

通过上面的伪代码可以看出，两种拓展属性存在有交集的可能性。所以我们做一种新的设计，但这种设计带来了类型的割裂。

我们分析下这种割裂感到底是什么。

1. 用户无法通过阅读得知 node 类型是什么。
2. 用户的注意常常会分散到 `ExtendsProps` 以至于忘记处理的数据核心是节点。

再向后设计，出现了一些设计变动，我们希望 derive 形似一个管道，可以对节点数据进行管道处理。
这时候 Derived 类型系统就无法支撑了，因为 `DerivedNode<D>` 没有赋值给下一个管道的 `PrimaryNode<P>` 的可能性。

重新思考过后，我决定对现有的类型做一些修改。

## 修改

```ts
interface INode {}

type Derive = <PNode extends INode,DNode extendsINode>(node:PNode):DNode=>{}

```

修改过后的类型变得更简单，继承关系更明确。

```ts
interface ICustomNode1 extends INode {}

interface ICustomNode2 extends INode {}

const derive: Derive<ICustomNode1, ICustomNode2> = (node) => {};
```

看起来 derived 概念被移除了，好像类型不完整了。但是并不是因为做了减法操作，而是设计方式的改变。

类型的设计重心被转移到了提示，而不是限制。

## 总结

设计类型时注意视觉感官，当从字面体会不到类型含义时，考虑是否可以修改为视觉友好的类型。

注意使用 Typescript 的出发点。不是为了限制和约束。反之而行，可能会设计出一个自己限制自己的的类型系统。

无法设计合理的类型时，及时的使用 any，而不是把自己禁锢在原地。（也许你的算法本来就有问题）

最后在 tsconfig 中打开 strict ，积极使用 IDE 的插件（比如 vscode 自带的 tsserver），尝试给自己更好的提示。
