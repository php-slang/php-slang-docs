# Why?
Functional programming became a buzzword of all software conferences in few last years.
PHP community does it's best to keep pace because there is a lot to gain.

What are the biggest benefits of functional programming?
- Less bugs
- Maintainable code
- Scalability out of box
- Easier testing
- Shorter code
- Easy to learn

You gain all of this by switching to FP (and PhpSlang will help you with it).
Take all of these advantages for granted at the moment - we will cover points above in a while.

## Short intro
Some people say that when you want to learn program functionally you have to forget everything you know about
programming and go back to school and your math teacher should be your programming master from this day.
There's a lot of truth in such thinking, but not completely - eg. OOP and FP create a great couple so all you know about
inheritance, composition etc. is still on the menu.
Staring to code purely functional does not discard all good practices like
[DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself),
[SOLID](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)),
[YAGNI](https://en.wikipedia.org/wiki/You_aren%27t_gonna_need_it) neither.

Even more - if you're familiar with [Domain Driven Design](https://en.wikipedia.org/wiki/Domain-driven_design) - you'll
see that functional thinking can be applied not only to specific pieces of code, but to the architectural decisions as
well. See this [great Greg Young's talk](https://www.youtube.com/watch?v=kZL41SMXWdM) for a reference (if you ask
yourself what event sourcing has to do with DDD check out Eric Evans
[book](https://books.google.pl/books/about/Domain_Driven_Design.html?id=hHBf4YxMnWMC&source=kp_cover&redir_esc=y)).

## The problem (with imperative programming)
### Every day headache
Does [this story](http://turnoff.us/geek/the-depressed-developer/) feel familiar to you?
Or bunch of other funny pictures about developers struggling with NullPointerExceptions or OutOfBoundExceptions?
There are tons of everyday small glitches in our projects. Sometimes such problems are resolved at code review when
other developers in your team find it, sometimes on production instance when angry clients start to call a support line.
We have to keep in mind lots of tiny dependencies that sum up to huge graph of connections which most developers have to
keep in mind when maintaining existing code.
The problem is that we are just humans and our buffer is quite tiny and [easy to flush](http://i.imgur.com/3uyRWGJ.jpg).

### Scalability issues
How many simultaneous threads with concurrent access to your data you're able to implement with full confidence?
2? 10? 20? What would you say about 100000? It is possible with pure functions. With shared mutable state
[not](http://henrikeichenhardt.blogspot.de/2013/06/why-shared-mutable-state-is-root-of-all.html).
And scalability is just one dimension of a whole problem.

### Mutability is counterintuitive
The business requirements tend to grow and change.
FP won't solve all your problems with complexity but will not create separate complexity on it's own and using immutable
data structures and monads will help you handle this complexity.

Imperative style can seem like a standard way of giving your computer instructions to process.
That's how computers work - instruction after instruction, are'nt they?
Well maybe (as long we're using Von Neumann computers) but it's definitely not how human brains work
and not how the world around us works.
Do you remember when the last time some physical object in your surrounding was just reasigned?

## The solution
### 1. Immutability!
The biggest difference between eg. Haskell and PHP is that you CAN NOT reassign values.
Keeping immutability will require a lot more discipline on your side when you use PHP,
but syntax in not important - thinking is!

From this day just keep in mind that:

**There is no such thing as variable!**

Forget about changing state of variables in random places.
All your code should be build of small logical blocks which are:

**Easy to reason about**

Every function should return the same result when run with the same set of input data. This way you don't have to think
about tons of dependencies inside your code. When you want to do something - you just run a function and you can be sure
that it will behave in the same way every time.

### 2. Ask a right questions
Imperative paradigm which is a standard for most of PHP applications leaves you with a code that describes HOW TO
achieve things.
Functional paradigm leaves you with a code that describes WHAT you want to achieve.

### 3. Function as a first class citizen
It must be mentioned because - well it's a functional programming.
Points below are rather a technical detail - but when you have these tools,
completely new possibilities are open for you.

- Currying - functions can return another functions as well.
- Higher order functions - functions can be passed as arguments to other functions.
- Anonymous functions - you can define a function just in a place you use them.
- Functions can be assigned to a variable (and object scope as well).

## Pros
### Less bugs
Immutability - again. Lets consider a following linear function you can remember from your childhood:

`f(x) = x * 2`

When x equals 2 you just know that the result of computation is 4. When 6 it's 12, when 7 it's 14 and so on.

More important than what this function does is what this function DOES NOT.
This function DOES NOT:
- set any kind of variables which are reused in next executions
- add new objects to globally available array
- create a temporary array to accumulate computed data

This is just a simple example, but it's actually an ultimate point of FP.

### Maintainable code
Code describes WHAT it does, not HOW. So you don't have to think that much on what was the intention of
a developer who created this piece of code - this information is already here.
Having a project written in a purely functional manner means also easier debugging and code navigation.
As long as all function calls are explicit, you'll see very little or no surprises when debugging.

### Scalability out of box
The biggest challenge of distributed architecture is concurrent access to data. You never know if a state is up
to date and who uses specific entities at the moment.
Concurrency creates tons of problems which can be solved using ideas like locks, mutexes, semaphores etc. but it
requires a lot of effort and creates new problems you have to keep in mind when extending existing solutions.

With **immutable** code and immutable data structures you can forget about all this stuff. When result of a function
is the same every time, there is no risk of running it in parallel on few CPU cores or even few physical machines.
You don't share a memory so code is not only more predictable but simply works faster.

Check [Future](../01_Usage/01_Essentials/06_Future.md) and 
[Parallel Collections](../01_Usage/04_Immutable_Data_Structures/02_Parallel_Collections.md)
for more information on this topic.

### Easier testing
No initialization. No tests for various states of dependent variables.
No side effects checks. Much less mocking and stubbing.

Just simple checks for return value and domain specific edge cases.
And again: [TDD](https://en.wikipedia.org/wiki/Test-driven_development)
and [BDD](https://en.wikipedia.org/wiki/Behavior-driven_development) are still on the menu.

### Shorter code
When you use nested function calls you end up with a much more condensed code.
Pipelined calls also tend to take less space and when everything is immutable,
most of your functions contain only one return statement - which appears in a first line of a method
(yes braces, semicolon and `return` keyword become to seem redundant - but unfortunately
you still have to use them in PHP).

Code condensation is visible especially for immutable collections - when you chain eg. `map()` and `filter()`
calls it becomes readable even for people with no developing skills.

### Easy to learn
One of the biggest myths of functional programming is that it has a big entry level.
Source of such a belief can be found in a track of life of an experienced software developers.
They are used to see small increments. When you develop with Zend for 3 years and then jump to Symfony - most of
concepts are the same, only approach to solve problems is different.
Functional programming requires you to really change the way you think (and it can be painful at the beginning), but
it's really not that much to learn (there's actually less things you have to learn to start compared to OOP).
When currying and monads give you a headache remember days when questions like
'what is a difference between interface and abstract class' were hard to answer for you - you have to
fill the gaps in your knowledge but from a perspective it's actually not that hard.

## OK, but my app has to store user input.
It can seem like a not so easy question to answer keeping in mind that all materials about FP repetitively convince
you not mutate data at all, but it does'nt mean that FP is limited only to transform data without side effects.
For an average PHP developer the simple answer is - store all your state changes in a database. That is the only place
where data changes should happen.

Immutability also does'nt mean that you cant create eg. database engine with purely functional language - check
Haskel's [STM](https://wiki.haskell.org/Software_transactional_memory) for a reference.
There's also a high chance that you are familiar with [RabbitMQ](https://www.rabbitmq.com/)
which is written in [Erlang](https://www.erlang.org/).
There is a lot of other software written in purely functional languages which are capable of storing varying data,
only approach to data changes is different.

## Show me the code

It's a prolixity without showing the code, so let's see this small example.
Both functions below return the same result for identical input. The only difference is used paradigm.

Imperative implementation:
```php
function averageOfNumbersSmallerThan(array $numbers, int $threshold) : string {
  $numbersSmallerThanThreshold = [];
  foreach ($numbers as $number) {
    if ($number <= $threshold) {
      $numbersSmallerThanThreshold[] = $number;
    }
  }
  $numbersDividedByThree = [];
  foreach ($numbersSmallerThanThreshold as $number) {
    $numbersDividedByThree = $number / 3;
  }
  $average = null;
  if (count($numbersDividedByThree) > 0) {
    foreach ($numbersDividedByThree as $number) {
      if (is_null($average) {
        $average = $number;
      } else {
        $average = ($average + $number) / 2;
      }
    }
  }
  if (is_null($average) {
    return 'It\'s impossible to count average for a given data.'; 
  } else {
    return 'Average is: ' . $average;
  }
}
?>
```

Functional implementation (using PhpSlang):
```php
function averageOfNumbersSmallerThan(ListCollection $numbers, int $threshold) : string {
  return $numbers
    ->filter(function($number) use ($threshold) { return $number <= $threshold; })
    ->map(function($number) { return $number / 3; })
    ->avg()
    ->map(function($avg) { return 'Average is: ' . $avg; })
    ->getOrElse('It\'s impossible to count average for a given data.');
}
?>
```

## Summing things up
Does an example above look convincing?
It's just a beginning. Remember that this example describes only one method and we're just scrapping the surface here.
We did'nt even touched the danger zones of exception handling, mutable object fields,
validation and any kind of storage.

The most important facts to remember before reading other materials on functional programming are:
- immutability - lets you deal with complexity and helps you create code which won't surprise you
- WHAT not HOW - this is the question that should be answered by your code
- treat function as a first class citizen - your system is composed of functions - they deserve some respect.

Have fun with functional programming!
