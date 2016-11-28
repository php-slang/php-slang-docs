## I read a documentation and did'nt found an answer.
PhpSlang treads unclear documentation as a bug.
If you have any issues with understanding described concepts or something is missing or misleading - feel free
to create an issue on [Github](https://github.com/php-slang/php-slang/issues)

## Where to start with functional programming?
Honestly - not with PHP. Give a chance to [Scala](https://www.scala-lang.org/), [Haskell](https://haskell-lang.org/),
or [Elixir](http://elixir-lang.org/).

All great ideas you'll learn out there will change the way you use PHP, but it requires a lot of discipline
to write immutable, purely functional code in PHP (and PhpSlang helps you here).

It can sound a little bossy but at the moment PhpSlang documentation seems to be the most comprehensive guide
on functional programming in PHP. There are of course other resources and they are listed on
[Other resources](../-00_General/02_Other_Resources.md) page.

## When not to use functional programming?
- Your default approach should be - write everything functionally.
I promise, when you learn to write purely functional code there is no coming back.
You'll want to write everything this way and when you'll face a problem which
requires different approach you'll see this.
There is no specific business domain where FP or traditional OOP is better. But keep
in ming that a gold hammer does'nt exist.
- There are two fields of a reality where there is a common belief that FP performs worse:
  - Embedded devices - when you write code for AVR or PIC microcontrollers and every bit of a
   memory matters - then it's probably best to stick with a classical C development.
  - Sorting algorithms - there's a whole story behind a quick sort implementations for
   a variety of functional languages and
   [benchmarks show](http://www.lambdadays.org/static/upload/media/14254629275479beameraghfuneval2.pdf)
   that because of a mutational nature of quick sort it performs worse when implementation in a purely functional way.

## How about things you can't implement with functional approach?
If you found such problem please let us know on [Gitter chat](https://gitter.im/php-slang/Lobby).

We'll be more that happy to help you solve the problem.
If you have such issue it can indicate some bugs or shortages of PhpSlang.

## Why some parts of PhpSlang are not purely functional?
It can seem like a lack of consequence, but keep in mind a main goal of a whole project.
It was created to give PHP developers tools that will
enable them to benefit from functional approach in a convenient way.
It hides some ugly parts of PHP. Those ugly parts are still there,
but with PhpSlang you don't have to deal with them every time you want to do something.

## How about mixing PhpSlang with other libraries?
PhpSlang does'nt change a behaviour of PHP nor adds any kind of special keywords.
It just provides helpful classes and functions.

It's still PHP so you can mix approaches.
If you want to have consistent code style - it's good to wrap/extend existing libraries with PhpSlang constructs
eg. `Option::of()` for all calls returning nullable results. 

## I use library X and it returns different results when used along with PhpSlang.
There is a high chance that one of two things happened:
- results of computations were wrong previously, but now with FP you fixed some bugs by an accident
(such magical things happen when you force yourself to formalize every aspect of an application)
- results of computations were correct previously, but now are wrong - it probably indicated non-thread-safe code.
Some libraries highly depend on mutations (even worse on mutations of static fields or globally accessible variables).
This way in a middle of computations in method A, you change a state needed by method B.
Please check if switching from ParallelListCollection (or other parallel collection) to ListCollection
won't solve your problem. TL;DR look for [race condition](https://en.wikipedia.org/wiki/Race_condition).

## I found a bug.
Place an issue on [Github](https://github.com/php-slang/php-slang/issues) so we know it exists.
It will be fixed ASAP.

## I want to use it in commercial project
Feel free - it's distributed under [MIT License](../-00_General/-00_License.md).

## When feature X will be available?
See [Roadmap](../-00_General/-01_Roadmap.md) for a reference.