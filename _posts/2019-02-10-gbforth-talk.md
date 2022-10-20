---
title: Game Boy hacking at FOSDEM
---

A week ago David and I were in Brussels, giving a talk about our first Amsterdam Hackers project at [FOSDEM 2019](https://fosdem.org/2019/). We explained how we developed a Forth cross-compiler for the original Game Boy by first reverse-engineering a `Hello World!` binary.

Our talk focusses on the incremental approach we used to hack on a device that we knew nothing about, and shows how well Forth fits into this process:

<div style="overflow: hidden; padding-bottom: 56.25%; position: relative; height: 0;">
<iframe src="https://www.youtube-nocookie.com/embed/g3HNhvW3lI8?start=0" allowfullscreen frameborder="0" style="left: 0; top: 0; height: 100%; width: 100%; position: absolute;"></iframe>
</div>

Although [gbforth](https://github.com/ams-hackers/gbforth) is far from complete, you can create terminal-based games (like the included Sokoban example) without much hassle already.

If you want dive deeper into Game Boy hardware, try these amazing talks:

- [The Ultimate Game Boy Talk](https://www.youtube.com/watch?v=HyzD8pNlpwI) -- Michael Steil
- [Reverse Engineering Fine Details of Game Boy Hardware](https://www.youtube.com/watch?v=GBYwjch6oEE) -- Joonas Javanainen
