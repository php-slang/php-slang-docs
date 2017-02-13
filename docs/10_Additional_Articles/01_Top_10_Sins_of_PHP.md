## Top 10 sins of PHP
PHP is a mature language yet it has it downsides especially in terms of functional programming (but not only).

Purpouse of a following article is pointing out some problems you'll face when you'll
start to code functionally (but not only).

**Short note**: *Following article compare PHP with other functional languages.
Scala is given as an example but this language was picket from tons of other languages because of its popularity.
Please don't take a false assumption that a basic meaning of this article is that PHP should become Scala.
PHP isn't Scala and will never be and probably should'nt become one. It has it's own upsides, but this article
is focused on downsides - especially in context of functional programming and gives
some answers on PhpSlang purpose and history.*

### 1. Verbose anonymous function syntax
When you start to program functionally with PHP you will create new functions - a lot.
And most of pure functions should contain only one return statement.
Let's take a look at a following example.

```
$users
  ->map(function(User $user) : string {
    return $user->getFirstName();
  });
```

All this simple piece of code does is transforming a list of users into a list of first names of those users.
Now let's slim down this example and cut all redundant information step by step.
So first - we don't need a `return` statement because pure functions perform only one operation or at least
we could assume that a result of a last statement in a function should be returned.
This is what most Scala, or even Ruby does. So this example could become:

```php
$users
  ->map(function(User $user) : string {
    $user->getFirstName();
  });
```

We also do not need a semicolons when we think about our code this way.

```php
$users
  ->map(function(User $user) : string {
    $user->getFirstName()
  })
```

Third part is `function` keyword. We know that `map()` should receive a function as a 1st parameter and what we
will have is still more than we need to know that this is a function definition.

```php
$users
  ->map((User $user) : string {
    $user->getFirstName()
  })
```

Next part of downsizing. Take a look at curly brackets. It's quite obvious that before arguments and returned
type definition there should appear actual implementation.

```php
$users
  ->map((User $user) : string $user->getFirstName())
```

Just look at it. This code could all fit in a single line of code.

And because we treat functions as immutable values for object we dont need brackets for
functions with arity 0 (functions that take 0 arguments).

```php
$users
  ->map((User $user) : string $user->getFirstName)
```

We could as well remove type annotations (but it's not a good idea as long as point 2 of this list is still valid).

```php
$users
  ->map(($user) $user->getFirstName)
```

But all those code examples is just a fantasy. With PHP till this moment we have to stick with:

```php
$users
  ->map(function(User $user) : string {
    return $user->getFirstName();
  });
```

This is the same functionality implemented with a Scala for comparision (yes it's a working code).

```scala
users
  .map(user => user.getFirstName)
```

### 2. No generic types
Lack of [generic types](https://en.wikipedia.org/wiki/Generic_programming) touches not only developers who want
to introduce functional programming. This is a problem of a whole community and everyone who
created non trivial OOP application feel this problem.

On 2016-01-06 there was introduced a proposal to implement this feature.
More about a proposal and shape of eventual implementation in PHP you can find at
[https://wiki.php.net/rfc/generics](https://wiki.php.net/rfc/generics)

[Fervo](https://www.fervo.se/) set a bounty for this feature:
[https://www.bountysource.com/issues/20553561-add-generics-support/backers](https://www.bountysource.com/issues/20553561-add-generics-support/backers)

You can ask yourself how such feature could change my life? Well lets consider you have want to pass a list of some
object to your method.

```php
function listTransformer(ListCollection $someList);
```

OK, so we have a list. But a list of what? There is no way to define such info.
You can (and should) of course try with describing content of iterables inside dockblock.

```php
/**
 * @param ListCollection|User[] $someList
 */
function listTransformer(ListCollection $someList);
```

So now we now that this parameter had to keep User entities.
Some tools will handle it somehow, some not. The worst part is that such annotation has completely no impact on
how our code will be executed. Dockblock annotation helps only for iterables - how about other
generic methods and classes? There is no easy answer at the moment. All this mess could be resolved when we could
write type definition similar to:

```php
function listTransformer(ListCollection<User> $someList);
```

Remember that generics are also composable so with generic types you could define more complicated structures.
Take a look at following example in Scala (you can achieve very similar things with Java or C++ as well).

```scala
def searchArticles(String searchQuery) : Either<List<Article>, SearchError>
```

Such definition in PHP at the moment is limited to:

```php
function searchArticles(string $searchQuery) : Either;
```

And you as a developer have to remember that Either should contain `ListCollection` of `Article`'s or `SearchError`
entity with detailed information about what happened wrong.

### 3. Convenient reassignments
When you decide to create purely functional code with PHP it will require a lot of discipline on your side.
And there is no way to stop other team members from breaking your convention.
There are no tools to prevent you from reassignments. Maybe some day PHPMD will be enriched with such functionality but
such features are'nt listed in it's roadmap at the moment.

Even worse - reassignments can occur even by a mistake.
Take a look at following example.

```php
$pleaseIDontWantToBecomeMutant = 'regular human';

$someList = [1, 2, 3];

foreach ($someList as $pleaseIDontWantToBecomeMutant) {
  //do nothing
}

echo $pleaseIDontWantToBecomeMutant;
```

What do you think what will be the value of a `$pleaseIDontWantToBecomeMutant`?

Hint: definitely not a `regular human`.

### Referential non-transparency
Next example. Some developer are very greedy in terms of memory (and they have their reasons).
You probably used a lot of 3rd party libraries or even build-in functions taking arguments via reference.

```php
$cow = 'cow';

$chernobylFunction = function(&$someArgument) {
  $someArgument = 'cow producing eggs';
  return $someArgument;
};

echo $chernobylFunction($cow);
echo $cow;
```

A sad result of this code is that `$chernobylFunction` will return  `cow producing eggs` as expected,
but a regular `$cow` will become `cow producing eggs` as well (and you'll see this problem on production
if you don't write enough tests). And our regular `cow` is lost - no more milk.

When you are a creator of such function you may think - I have it all under control.
The problem is that users of that function don't.
As a library user you have to be very careful and check every function definition if it's arguments are passed
by a copy or a reference.

### 4. Inconvenient cloning
Take a look at this class.

```php
class House {
  /**
   * @var string
   */
  private $size;

  public function __construct(string $size) {
    $this->size = $size;
  }

  public function getSize() : string {
    return $this->size;
  }

  public function setSize(string $newSize) : House {
    $this->size = $newSize;
    return $this;
  }
}
```

Nothing special isn't it? Regular class with getters and setters like we're used to have.
Method `setSize` returns `$this` so we have a convenient way to chain few method for one object.

Let's consider a following usage.

```php
$smallHouse = new House('small');
$bigHouse = $smallHouse->setSize('big');

echo $bigHouse->getSize();    //writes 'big'
echo $smallHouse->getSize();  //writes 'big' as well :(
```

What happened? We mutated a state again - somehow by an accident.
OK, so you transform your code so it's purely functional.
Let's say that You already read a PhpSlang documentation and you know that every function name with `set` as a prefix
should become a similar function with `with` as a prefix. How to achieve it?
If you used Scala such method would look somehow like:

```scala
case class House(size:Int) {
  def withSize(newSize : Int) = copy(size = newSize)
}
```

PHP does not provide any build in features to clone objects with only selected fields changes, so you
end up creating completely new objects copying all fields to constructor and changing those values
you're interested in or with class which looks like this:

```php
class House {
  /**
   * @var string
   */
  public $size;

  public function __construct(string $size) {
    $this->size = $size;
  }

  public function getSize() : string {
    return $this->size;
  }

  public function withSize(string $newSize) : House {
    $clone = clone $this;
    $clone->size = $newSize; //note that $size field had to be changed to public
    return $clone;
  }
}
```

Does'nt look very beautiful is it?

PhpSlang's [Copy trait](../01_Usage/01_Essentials/01_Copy_Trait.md) partially resolves this issue and
allows you to write something like:

```php
class House {
  use Copy;

  /**
   * @var string
   */
  private $size;

  public function __construct(string $size) {
    $this->size = $size;
  }

  public function getSize() : string {
    return $this->size;
  }

  public function withSize(string $newSize) : House {
    return $this->copy('size', $newSize);
  }
}
```

And now you can use this class in a following way.

```php
$smallHouse = new House('small');
$bigHouse = $smallHouse->withSize('big');

echo $bigHouse->getSize();    //writes 'big'
echo $smallHouse->getSize();  //writes 'small' as expected :)
```

### 5. Very little syntactic sugar

#### No placeholder syntax

#### No partially applied functions

#### No implicit scoping

#### No wildcard import

#### No pipeline operator
https://wiki.php.net/rfc/pipe-operator

### 6. Imperative ecosystem

### 7. Lack of Closure descriptions

### 8. No pattern matching

### 9. No macros

### 10. No standard for annotations
