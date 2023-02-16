---
title: Why Play Games When You Can Write a Game Boy Compiler Instead?
footer: 👾
excerpt_separator: <!--more-->
---

> Originally appeared on the [Reaktor blog](https://www.reaktor.com/blog/why-play-games-when-you-can-write-a-game-boy-compiler-instead/)

When was the last time you had to worry about how much memory your code could
use? Have you ever had to count the clock cycles it takes to render an animation
on the screen? <!--more--> If you're a developer like me, you probably spend most of your
time working with a modern, higher-level programming language. JavaScript,
Python, Clojure... take your pick: with few exceptions, you extensively rely on
the advancements computer science has booked over the last few decades.

Ever felt like things got a little bit too easy?

Well, you're in luck!

We found an excellent way to break up your daily grind --- by diving into the world of retro computers (they are just like real computers, only worse!). So whip out a couple of AA batteries and blow into those dusty cartridges. That's right, it's time to write a compiler for the Game Boy!

## The Usefulness of "Useless" Projects

Amsterdam Hackers was formed as a Reaktor hobby club for people who enjoy
tinkering with software projects without being too worried about the usefulness
of those projects. Rather, we try to learn by doing and appreciate the process
of discovering knowledge hidden in complex topics. A lot of people underestimate
their abilities when they encounter problems they do not know how to solve yet.
Amsterdam Hackers encourages people to work on these types of challenges so they
can overcome that initial fear of not knowing.

With that in mind, we brainstormed about a suitable first project and kept
coming back to the topic of Game Boys. There's probably a good reason for that:
for most of us, these little devices pack a ton of nostalgia --- holding one is
like being transported back into childhood. And yet, even though we were a bunch
of experienced engineers, we knew virtually nothing about how to program them.
Needless to say, we were eager to find out! Of course, simply writing a small
game would not have been juicy enough, so we decided to develop a programming
language for the game console. And then, maybe build a game on top of _that_ while
we're at it? Now we're talking!

## Doing Things the Hard Way

Some 30 years after its initial release, you can find a lot of specs, tools, and
tutorials on how to make a Game Boy game from scratch. However, it's often a bit
sluggish, and it takes a long time before you see anything interesting on the
screen --- until then, you're stuck with reading pages and pages of documentation.
Not a very appealing thought. Plus, the whole point was to discover and learn
these things for ourselves. So we figured we could take a more unorthodox
approach: start with a working game and reverse-engineer our way back from
there. By working bottom-up instead, we would have a short feedback cycle and
ensure a functional version of our project between each commit.

![A screenshot of a Game Boy emulator displaying the text "Hello World !"](/assets/img/example.png)

After some searching, we found a suitable ROM to kick our project off with: An
example "game" that simply displays "_Hello World !_" on the screen. Perfect! We
peeked at the contents of the file using `hexdump` and were greeted with a lot
of (at the time meaningless) numbers:

![The hexdump output of the first 160 bytes of a file named hello.gb](/assets/img/hex.png)

From there on, we started investigating what all those bytes meant and began
deciphering the program 8 bits at a time.

## Going Forth

Our tool of choice? An unusual programming language called Forth. It's a
stack-based language with almost no syntax: everything is either a number or a
word (essentially a function), and both are delimited by spaces. There are no
characters with special meanings, or reserved keywords like most other languages
have, so words can be named whatever you want. This lack of syntactical
constructs makes learning Forth deceptively simple but also allows --- and almost
requires --- you to build your own "language" on top of it. You can then
simultaneously work with very low-level code and more abstract procedures.
Perfect for this project!

After getting familiar with the basics of Forth, we created the very first
version of our compiler:

![A screenshot of Forth code, defining a Main word](/assets/img/rom.png)

The numbers you see here are copied directly from the `hexdump` output, and we
postfixed each byte with the c, word (which writes the byte to a file). The
result is a program that emits the exact same bytes from our Hello World demo to
a new file. Of course, this hardly qualifies as a real compiler yet --- it
generates the same output every time you run it, after all --- but it was written
in Forth and produced a functional ROM, providing a solid basis for the project.

We then started grouping the raw bytes into more readable chunks. Instead of a
single `main` word, we split our compiler into `emit-header` (which primarily
contains metadata about the game) and `emit-game` (containing the executable
code). The header then got divided further into `emit-title`, `emit-logo`,
`emit-version`, and so on. Gradually the code became more legible, even if, down
the line, all of these words were still spitting out the same bytes as before.

The neat thing about this approach is the incremental nature of the work. At any
given point, we had a functioning "compiler," and we could easily verify that
the generated ROM remained the same after each change.

Having the bytes neatly organized allowed us to make the first significant
improvement to the ROM itself. We were now able to update the message that got
printed to the display by changing the ASCII values `57 6f 72 6c 64` to, let's
say, `52 65 61 6b 74 6f 72` perhaps?

![A photo of a yellow Game Boy displaying the text 'Hello Reaktor !'](/assets/img/reaktor.jpg)

_Hello indeed!_

## Piecing Together the Assembler

After figuring out the cartridge header, we continued decoding the rest of the
ROM. Every byte (or combination of bytes) in the output corresponds to an opcode
in assembly. Translating them is as easy as looking up the opcodes from a
reference table and defining a new Forth word for each instruction:

![A screenshot of Forth code, defining words for the NOP, JMP and HALT assembly instructions](/assets/img/asm.png)

Luckily our _Hello World_ demo contained less than 100 instructions (and most were
duplicates), so this step went by reasonably quickly. With a bit more
refactoring and applying some binary math, we completed a basic assembler and
added the rest of the instruction set that the ROM did not use.

## Crafting the Compiler

With our example ROM fully decompiled into assembler instructions, it was time
to crank things up a notch: turning the assembly code into Forth code. We first
had to create a kernel for the run-time to achieve this. Since Forth is a
stack-based language, this essentially meant implementing a stack and a couple
of subroutines to manipulate that stack in assembly.

![A screenshot of Forth code, defining the words Clear-Stack, Push-D and Pop-E using assembly](/assets/img/kernel.png)

It takes a little while to get into the assembler mindset, but these pieces of
code were still relatively small and self-contained, so you don't get too
overwhelmed by the tasks ahead if you take it one step at a time. The actual
kernel we wrote looks a bit more convoluted to support 16-bit values (with the
hardware being 8-bit, which means juggling double the number of registers), but
the essence remains the same.

Having this bare kernel allows the creation of our primitives --- the very first
words in the "core library" that will still have to be written in assembly. This
includes many fundamental Forth words, such as:

- Stack manipulation like `DUP` (duplicates the top item on the stack).
- Arithmetic like `+` (adds the top 2 items on the stack together).
- Memory access like `C@` (reads a byte from the address specified by the top of
  the stack).

![A screenshot of Forth code definitions for Dup, Plus and C-at](/assets/img/core.png)

The primitives are very small and self–contained because they always interface
through the parameter stack. After some initial struggle, creating a core
library with the most basic operations is relatively straightforward.

The final step that remains is the Forth compilation itself. I won't go into all
the details and intricacies here, but the minimal viable version only requires a
few elements:

1. Load the user's source code and start reading piece by piece.
2. Whenever you encounter a number, emit the assembly to push that number to the
   stack.
3. Whenever you encounter a word, find the address of its definition and emit a
   `CALL` instruction to it.

Once the compiler and core library were in place, we could now rewrite our
_Hello World_ demo using Forth exclusively. This, in turn, also spawned other
reusable high-level libraries that developers can include to run common procedures.
What started as a bunch of unreadable bytes was now reduced to a mere 8 lines of code:

![A screenshot of the full Hello World program written in Forth](/assets/img/hello.png)

While Forth is a language that requires a very different way of thinking about
your code, the close proximity to the hardware while still allowing you to
create high-level constructs makes it really interesting. We could seamlessly
translate all assembly code into Forth and gradually worked our way up to
increasingly abstract words like `clear-screen` and `load-font`.

## Can I Run It?

Time to see if our Forth implementation is actually worth its salt! To benchmark
our compiler, we found a suitable game (written in Forth) that we could try to
run on an actual Game Boy. This required a bit of extra effort to support the
remainder of the standard Forth dictionary, but soon enough, we were playing
Sokoban on that iconic dot matrix screen:

![A photo of a grey Game Boy running Sokoban](/assets/img/soko.jpg)

What a beautiful sight! A game that someone had originally written for a desktop
Forth --- years before we even started our project --- had now been successfully
cross-compiled into a playable Game Boy ROM.

## The Journey is the Reward

At the end of all this, we released gbforth --- a Forth compiler for the original
Game Boy. It might be a bit rough around the edges, but is it usable?
Definitely! A few example games were made with gbforth already, and it's
honestly a lot of fun to develop games this way compared to the typical C-based
toolkits available.

Apart from developing the compiler itself, we learned about a lot of esoteric
topics in the process:

- Programming in Forth while recreating Forth for a new (old?) system
- Learning about the Game Boy's architecture and historical hardware hacks
- Brushing up on binary arithmetic to write math operations in assembly by hand
- Setting up Forth for our CI pipeline to automate end-to-end testing on an
  emulator
- Reliving the struggles of game development in the early 90s in general

I obviously don't claim complete expertise in any of these fields. However, I
can still comfortably introduce you to metaprogramming in Forth, the trade-offs
of subroutine threading vs. direct threading on a Z80 processor, and how your
favorite Game Boy game achieves the fade-in effect at the menu screen.
Furthermore, gbforth landed our first international conference talk at FOSDEM in
Brussels.

![A photo of Tijn and David doing a conference talk](/assets/img/fosdem.jpg)

Working on gbforth reminded us that sometimes the journey itself is the reward.
We could have opted for an easier path early on, but then we would have missed
out on all of these hidden gems we discovered along the way. Instead, we got to
learn some interesting stuff, bond with each other, and see that even when you
have no clue how to do something at first, by breaking it apart, you are able to
figure it out!

## Further reading

- [Project page for gbforth](https://gbforth.org)
- [Starting FORTH](https://www.forth.com/starting-forth/1-forth-stacks-dictionary)
- [The Ultimate Game Boy Talk](https://www.youtube.com/watch?v=HyzD8pNlpwI)
- [Game Boy Pan Docs](https://gbdev.io/pandocs)

You can also listen to an episode where we [talked about gbforth](https://www.reaktor.com/forkpullmergepush/finding-the-corner-of-the-internet-you-love-with-amsterdam-hackers) on Reaktor's [Fork Pull Merge Push](https://www.reaktor.com/forkpullmergepush) podcast.
