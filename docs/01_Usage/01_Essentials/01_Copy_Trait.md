# The problem
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

It does not look suspicious as long as you don't use it.

```php
$smallHouse = new House('small');
$bigHouse = $smallHouse->setSize('big');

echo $bigHouse->getSize();    //writes 'big'
echo $smallHouse->getSize();  //writes 'big' as well :(
```

Why it works like that? Setters are not *referentially transparent*. Those methods mutate a state of an application and you, as a developer, have to remember about existence of such actions taken internally by methods you call.


## The naive solution

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

Using a methods named `with*` instead of `set*` is a step forwards, but this solution still has some disadvantages.

1st - you have to write those 3 lines every time you want to get an object with value changed.
2nd - you have to make changed field public because when you clone `$this` and try to modify some field you don't have an access to `private` and `protected` fields anymore. Leaving those fields with such liberal property visibility may lead to a lot of troubles.

# The solution - Copy trait
```php
use PhpSlang\Util\Copy;

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

`Copy` trait made a solution from naive version much shorter and you don't have to change property visibility.

Disadvantages?

Well - almost none, but if you need "getters" and "setters" (in this case "with'ers") it may be a sign that you didn't yet switch your thinking to functional.
You should rather avoid changing a state of any value. Let's take a look a how you probably should deal with derivates of existing objects.

# The solution - Immutable DTO
DDD teaches us that any value object is immutable and it comes hand in hand with functional programming .
```php
class House {
  /**
   * @var string
   */
  private $size;

  /**
   * @var float
   */
  private $someOtherField;

  public function __construct(string $size, float $someOtherField) {
    $this->size = $size;
    $this->someOtherField = $someOtherField;
  }

  public function getSize(): string {
    return $this->size;
  }

  public function getSomeOtherField(): float {
    return $this->someOtherField;
  }

}
```

This class does not contain anymore: `setters`, `Copy` trait, `with'ers`, `cloning`.
How to use it properly?

```php
$smallHouse = new House('small', 14.234);
$bigHouse = new House('big', $smallHouse->getSomeOtherField());

echo $bigHouse->getSize();    //writes 'big'
echo $smallHouse->getSize();  //writes 'small' as expected :)
```

What changed? Almost nothing. You still create a new instance of class `House` to create a value named `$bigHouse`. And you still create this value depending on already existing object `$smallHouse`, but this time it's more expressive and writting a code this way will probably make you think about overal structure of your solution in a way that you tend to use constructor just once, but with a well precalculated data instead of changing a values here and there.