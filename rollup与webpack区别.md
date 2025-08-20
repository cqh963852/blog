我在前端领域已经工作了很多年，rollup和webpack我同时都使用过。这篇文章，我想聊一聊 rollup 和 webpack 之间的一些区别。

先说一说两个工具的一些相同点，大部分用户都会使用这两个工具来做相同的事情————前端工程打包。

但是这两个工具在定位上还是有一些差距的。webpack 的年纪比 rollup 更大，所以我们可以通过发展历史去看一下。

在 webpack 之前，我比较熟悉的工具是 glup。它的核心目的是增强工程师的工作流。在前端工程师中有一些很重要的工作流，热更新，打包。

webpack 相比 glup 规范了打包热更新的流程，让流程可以配置化，而不是用编码定义流程。

rollup 的主要目标是代码转换。

> Compile small pieces of code into something larger and more complex

有一段时间我尝试去编写一些插件。这个插件会根据一个 JSX 文件生成一个 JSX 和 CSS 文件。然后我期望将这些文件进行打包。

在这个过程中我发现一个问题。如果我想要在 webpack 实现一个这样的Loader。并且相关的文档是缺失的。我并不知道如何通过插件来生成多个文件。

其中比较明显的一个部分是当一个文件需要被转换成多个文件的时候，在 webpack 中这一需求的编码是非常复杂的。并且是文档缺失的状态。https://github.com/webpack/webpack/discussions/16494

而 rollup 处理这一部分则相对的比较简单。你可以通过 rollup 的一些 API 生成这些文件。然后再重新调用 rollup 的主流程对他们进行打包工作。

在前端技术栈逐渐复杂的当下，我认为这样的特性是非常重要的。这也是为什么 rollup 2025年的下载量超过了 webpack。
