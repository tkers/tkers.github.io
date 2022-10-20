---
title: Codemods Talk
---

Last week I gave my [first tech talk](https://www.youtube.com/watch?v=xS7UrNPmYX8)
at a JavaScript meetup in Amsterdam. While it was a bit daunting at first, I luckily have amazing colleagues who helped me a lot when preparing this talk (and managed to convince me that I have something worth sharing in the first place).

A 30 minute talk is far too short to give a highly detailed explanation on writing code transformations, but hopefully serves as a nice introduction:

<div style="overflow: hidden; padding-bottom: 56.25%; position: relative; height: 0;">
<iframe src="https://www.youtube-nocookie.com/embed/xS7UrNPmYX8?start=5" allowfullscreen frameborder="0" style="left: 0; top: 0; height: 100%; width: 100%; position: absolute;"></iframe>
</div>

If you want to play with codemods yourself:

- [JSCodeShift](https://github.com/facebook/jscodeshift) provides an API to write code transformations, and a CLI tool to apply them to your code base.
- [AST Explorer](https://astexplorer.net) is a great tool to discover what an Abstract Syntax Tree (AST) looks like, and also lets you write codemods in the same screen (which makes writing and testing them a breeze).
- [reactjs/react-codemod](https://github.com/reactjs/react-codemod) and [cpojer/js-codemod](https://github.com/cpojer/js-codemod) contain a lot of useful codemods that can help get you started.
