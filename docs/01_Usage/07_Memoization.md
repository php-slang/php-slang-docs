# Problem/possibility

One of advantages (and principles) of functional programming is referential transparency. Ite means that a function run with an identical set of parameters should always return identical result. Putting things this way we can fast endup with a conclussion that there is no sense in complete execution of methods that were already used at leas once with the same set of parameters. This makes sense especially if computations performed by a given method are resource exhaustive.

Consider a following example:
```php
class NonTrivialComputations {
  public function heavyComputations(string $name): int {
    //it doesn't matter what this function does, but we know that:
    // 1 - it's pure function (no side effects, no external resources used)
    // 2 - computations may take a long time and we want to optimize this part
  }
}
```

Running such code will always lead to executing *heavyComputations*.

## Solution

PHP does not give us any kind of tools to make this part easy (eg [lazy val](http://docs.scala-lang.org/overviews/core/value-classes.html)). To make your methods memoizable - you have to perform a little gymnastics:

```php
namespace PhpSlang\Memoisation\Memo;

class NonTrivialComputations {
  /**
   * @var Closure
   */
  public $heavyComputations;

  public function __construct()
    {
        $this->heavyComputations = (new Memo)->memoize(function (string $name): int {
          return $this->heavyComputations($name);
        };
    }

  private function heavyComputations(string $name): int {
    //it doesn't matter what this function does, but we know that:
    // 1 - it's pure function (no side effects, no external resources used)
    // 2 - computations may take a long time and we want to optimize this part
  }
}

```

Our original method is private now and we created a class field of the same name which is publicly accessible.

Then we assigned to this field a result of `(new Memo)->memoize` method which takes a new function as a parameter and this function has identical set of parameters as `self::heavyComputations` so we only pass a call.
What we actually produced is a class with an object inside which will run our methods as previously, but when it's called again - already computed data will be returned.

## Dangerzone 1

