## What this article is about and what is'nt?
This part of PhpSlang documentation covers basic syntax constructs given by PHP that are crutial in terms of functional proogramming.

This article also display what is possible in PHP with functionall approach and what is'nt.

This is not a tutorial on how to program functionally and/or with a PhpSlang library.

## Minimal survival resources
*Functional programming is a style. It works in any programming language.* Brian Beckman

This is a great quote, especially in case of PHP which usually isn't a first thought when you talk about FP.

Well this quote isn't true for every programming language. If you want to program functionally you need one crucial feature in a language: **Function is a fisrt class citizen**.

What does it mean?

- can be stored in variables and data structures
- can be passed as a parameter to a subroutine
- can be returned as the result of a subroutine
- can be constructed at runtime
- has intrinsic identity (independent of any given name)

In other words: you can assign function to "variable", return customized function from another function and take as an argument using standard language constructs with no need to rely on reflection.

Does PHP fulfill all listed requirements? - YES... well almost.

To have a better picture, take a look at [this table at Wikipedia](https://en.wikipedia.org/wiki/First-class_function)

We see that two bits of a puzzle are missing. We'll go back to it later in this article.

## Function definition
Just to remind. The simplest example as it only can be:
```php
public foo($a) {
  return $a * 2;
}
```

This function returns parameter $a multiplied by 2.

PHP gives us ability to typehint parameters taken by method and it's result so in this case it's safer (and recommended) to explicitly describe types:
```php
public foo(int $a): int {
  return $a * 2;
}
```

In PHP functions can be defined as part of classes - in that case we call them *methods*:

```php
class Bar {
  public function foo(int $a): int {
    return $a * 2;
  }
}
```

Things above are nobrainers. Every PHP developer arount the world uses those contructs everyday. But now it get's interesting.

## Function as a parameter

Let's consider a following function (regartheless if it's just a dangling function or class method).

```php
function doSomethingFunnyWith(string $originalTxt, int $times, Closure $transformation): string {
  $accumulator = '';
  for($i=0; $i < $times ;$i++) {
    $accumulator .= $transformation($originalTxt);
  }
  return $accumulator;
}
```

This function:
- has no side effects
- contains variable *$accumulator* which is a bad practice in perspective of FP (and also when you thing about scalability), but we'll cover this topic later
- take 3 parameters:
  - some string to play with
  - numeric value to indicate how many times we want to repeat transofmed text
  - Closure (method) which is then used to transform $originalTxt

When function takes another function as an argument we call it a **higher-order function**.

Look closer at this part: `$transformation($originalTxt)`
Function parameter named: `$transformation` is a function so we use it as a function, simply adding brackets and passing inside them function parameter(s) as we would do for every other function.

How to use such function? String and int parameters don't require explanation but Closure needs some more attention because we rules and possibilities vary between different ways you use such function.

### Using higher order functions
PHP gives you a way to define function just in a place you want to use it. Such functions are called **anonymous functions**. You don't give a name to such function, there is no way to reuse it in other parts of a code but it's also a very comfortable and popular way to define functions (and with FP approach you're going to write a lot of them). So if you'd like to print on screen "D_O_OD_O_OD_O_O" you could execute *doSomethingFunnyWith* in a following way:

```php
echo doSomethingFunnyWith(
  "DOO",
  3,
  function(string $inp): string {
    return implode('_', str_split($inp));
  }
);
```

But now let's consider a situation when you actually would like to define this function once and have an ability to reuse it.

```php
function underscoresEverywhere(string $inp): string {
  return implode('_', str_split($inp));
}
```
and now let's use it:
```php
echo doSomethingFunnyWith(
  "DOO",
  3,
  underscoresEverywhere
);
```
hmm...
```
PHP Notice:  Use of undefined constant underscoresEverywhere - assumed 'underscoresEverywhere' in ... on line ...
```
Well maybe:
```php
echo doSomethingFunnyWith(
  "DOO",
  3,
  underscoresEverywhere()
);
```
aahhh, again error:
```
PHP Fatal error:  Uncaught TypeError: Argument 1 passed to underscoresEverywhere() must be of the type string, none given, called in ...
```

Quite frustrating huh?

Well PHP isn't a perfect language for FP. When you want to: name a function AND be able to pass it as a parameter you have to assign it to a "variable" (the word variable is written with quotes because in FP you should rather think in terms of immutable values rather than variables).

Do you remember [wikipedia listing](https://en.wikipedia.org/wiki/First-class_function)? Well - this is the reason why section **Nested functions** -> **Named** is marked as *yellow -> Using anonymous*.

So what is a propper way to use such function?

#### Version 1a
As said previously - assign function to "variable":
```php
$underscoresEverywhere = function (string $inp): string {
  return implode('_', str_split($inp));
};//don't forget about semicolon :)

echo doSomethingFunnyWith(
  "DOO",
  3,
  $underscoresEverywhere
);

```
#### Version 1b
Same approach but inside a class:

```php
class Foo {
  /**
   * @var Closure
   */
  $underscoresEverywhere;

  public function __construct() {
    $this->underscoresEverywhere = function (string $inp): string {
      return implode('_', str_split($inp));
    };
    //BAD NEWS - passing methods inside class sucks too. If you want to make method "passable", you
    // have to assign it to class field.
  }

  private function doSomethingFunnyWith(string $originalTxt, int $times, Closure $transformation): string {
    $accumulator = '';
    for($i=0; $i < $times ;$i++) {
      $accumulator .= $transformation($originalTxt);
    }
    return $accumulator;
  }

  public function testRide(): string {
    return $this->doSomethingFunnyWith(
      "DOO",
      3,
      $this->underscoresEverywhere
    );
  }
}

echo (new Foo())->testRide(); //remember about additional brackets around constructor usage, so you can chain method calls with no need to use variables
```


#### Version 2a
You can also wrap every named function/method with the anonymous function:

```php
function underscoresEverywhere(string $inp): string {
  return implode('_', str_split($inp));
}

echo doSomethingFunnyWith(
  "DOO",
  3,
  function(string $inp): string {
    return underscoresEverywhere($inp);
  }
);

```


#### Version 2b
Same as version 2a but for a class definition:
```php
class Foo {
  private function underscoresEverywhere(string $inp): string {
    return implode('_', str_split($inp));
  }

  private function doSomethingFunnyWith(string $originalTxt, int $times, Closure $transformation): string {
    $accumulator = '';
    for($i=0; $i < $times ;$i++) {
      $accumulator .= $transformation($originalTxt);
    }
    return $accumulator;
  }

  public function testRide(): string {
    return $this->doSomethingFunnyWith(
      "DOO",
      3,
      function(string $inp): string {
        return $this->underscoresEverywhere($inp);
      }
    );
  }
}

echo (new Foo())->testRide();
```

It may seem like this notation is quite verbosive, but it's actually a most popular and comfortable way use higher order functions in PHP.
Well - [PHP syntax does not spoil](../10_Additional_Articles/01_Top_10_Sins_of_PHP.md).

## \Closure vs \Callable
One more way to implement the example we iterate with, is to use `Callable` type for `doSomethingFunnyWith` definition instead of `Closure`.
```php
class Foo {
  private function underscoresEverywhere(string $inp): string {
    return implode('_', str_split($inp));
  }

  private function doSomethingFunnyWith(string $originalTxt, int $times, Callable $transformation): string {
    $accumulator = '';
    for($i=0; $i < $times ;$i++) {
      $accumulator .= $transformation($originalTxt);
    }
    return $accumulator;
  }

  public function testRide(): string {
    return $this->doSomethingFunnyWith(
      "DOO",
      3,
      [$this, 'underscoresEverywhere'] //now you can pass a method name with a string
    );
  }
}

echo (new Foo())->testRide();
```

`Callable` may seem like a default choice but it's actually more bug prone variant. `Callable` gives you some more flexibility, but with great power comes a great responsibility. Let's consider a following code example.

```php
class Foo {
  private youDontWantToRunThisMethod(): void {
    launchNuclearMissiles();
    dropDatabase();
  }

  private function underscoresEverywhere(string $inp): string {
    return implode('_', str_split($inp));
  }

  private function doSomethingFunnyWith(string $originalTxt, int $times, Callable $transformation): string {
    $accumulator = '';
    for($i=0; $i < $times ;$i++) {
      $accumulator .= $transformation($originalTxt);
    }
    return $accumulator;
  }

  public function testRide(string $methodName): string {
    return $this->doSomethingFunnyWith(
      "DOO",
      3,
      [$this, $methodName]
    );
  }
}

$userInput = 'underscoresEverywhere';
echo (new Foo())->testRide($userInput);

$userInput = 'youDontWantToRunThisMethod';
echo (new Foo())->testRide($userInput);
```
:)

As you see - using strings to indicate which methods you want to use is a bad idea. Your IDE will have problems with proper types resolving, so you won't have code completions, so your live standard will drop and then everybody dies. Same rules apply to [call_user_func](http://php.net/manual/en/function.call-user-func.php) method usage - you should avoid it at all costs as long as it's possible.

For more info about differences between `Closure` and `Callable` please reffer to PHP official documentation [\Callable](http://php.net/manual/en/language.types.callable.php) & [\Closure](http://php.net/manual/en/class.closure.php)

## Function as a result

One more part that needs our attention is returning new functions from functions. Such mechanism is called **currying**.
Let's rewrite again our method presenting "D_O_OD_O_OD_O_O" string. Let's say you'd like to customize letter used between characters, but the rest of a method should stay the same. Obvious and easiest method would be to just add another `string` parameter, but let's try another way.

```php
class Foo {
  //that is a place where method underscoresEverywhere was located
  private function curriedStringFiller(string $glueWith): Closure {
    return function(string $inp) use ($glueWith): string {
      return implode($glueWith, str_split($inp));
    };
  }

  private function doSomethingFunnyWith(string $originalTxt, int $times, Closure $transformation): string {
    $accumulator = '';
    for($i=0; $i < $times ;$i++) {
      $accumulator .= $transformation($originalTxt);
    }
    return $accumulator;
  }

  public function testRide(): string {
    return $this->doSomethingFunnyWith("DOO", 3, $this->curriedStringFiller('+'));
  }
}

echo (new Foo())->testRide();
```

This time you are going to end up with: "D+O+OD+O+OD+O+O"

What happens in this example? Method `curriedStringFiller` returns another methos - it's anonymous function created at runtime.
You can see one more PHP syntax keyword used which is `use`. You have to use such syntax to be able to use "variables" from outside scope inside your new anonymous methods (without `use ($glueWith)` you couldn't see content of this "variable").

Let's also look closer at lines:
```php
return $this->doSomethingFunnyWith("DOO", 3, $this->curriedStringFiller('+'));
```
Remember that `$this->curriedStringFiller('+')` actually does nothing with a string passed to `doSomethingFunnyWith`. It only returns a new function which is passed to `doSomethingFunnyWith` and then method `doSomethingFunnyWith` in it's internal implementation uses a resulting method to transform a `string` given as a first parameter.

What we did in presented example could be written in a much shorter way if we used languages like JavaScript, Scala or Haskell. Those languages have build in mechanisms for currying and/or *partially applied functions** (every function can be used as curried - when you don't provide all required parameters to function, a new one is created and this new function requires only missing parameters from original one).

Again - refer to [Wikipedia listing](https://en.wikipedia.org/wiki/First-class_function) for more info. This is a part where PHP lacks some basic functionality. There is just no such thing as **Partial application** in PHP. You may like to see what you loose that exists for [Haskell](https://wiki.haskell.org/Partial_application), JavaScript [1](https://lostechies.com/derickbailey/2012/07/20/partially-applied-functions-in-javascript/), [2](http://raganwald.com/2015/04/01/partial-application.html) or Scala [1](http://docs.scala-lang.org/tutorials/tour/currying.html), [2](http://docs.scala-lang.org/cheatsheets/). But there is also a good news! You can live without this feature. When you'd use a partially applied function, you can create a new function instead (as presented in the example above, where we "manually" created a new function).

## One more last step

Aa it was mentioned in introduction - this article covers PHP syntax elements required to be able to develop in a functional style, but may not display good practicies. Functional programming crutial principle is immutability. At the same time you could see that in code example we use `for` statement with variables like `$i` and `$accumulator`. Let's add some final touches to the last version of our class to make it purely functional.

```php
  private function doSomethingFunnyWith(string $originalTxt, int $times, Closure $transformation): string {
    return array_reduce(range(1, $times), function(string $accumulator, int $item) use ($transformation, $originalTxt): string {
      return $accumulator.$transformation($originalTxt);
    }, '');
  }
```

What happened to doSomethingFunnyWith method?
We used `range` method to generate list of `$times` elements, and then we used `array_reduce` method to iterate over range and build a final string (ignoring an `$item` variable, because we don't actually need it).

With PhpSlang a final form of this class may look like this:

```php
use PhpSlang\Collection\ListCollection;

class Foo {
  private function curriedStringFiller(string $glueWith): Closure {
    return function(string $inp) use ($glueWith): string {
      return implode($glueWith, str_split($inp));
    };
  }

  private function doSomethingFunnyWith(string $originalTxt, int $times, Closure $transformation): string {
    //voilà - functional, referentially transparent, readable, chainable, very little or no error risk
    return (new ListCollection(range(1, $times)))
      ->fold('', function(string $accumulator, int $item) use ($transformation, $originalTxt): string {
        return $accumulator.$transformation($originalTxt);
      });
  }

  public function testRide(): string {
    return $this->doSomethingFunnyWith("DOO", 3, $this->curriedStringFiller('+'));
  }
}

echo (new Foo())->testRide();
```