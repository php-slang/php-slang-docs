# The problem

When you develop in a functional style you tend to create a lot of small functions than can be composed.

Lets consider a following example:

```
class ALotOfNestedFunctionCallsExample {

  //it doesn't matter what those functions do internally
  abstract protected function transformation1(TypeA $input): TypeB;
  abstract protected function transformation2(TypeB $input): TypeC;
  abstract protected function transformation3(TypeC $input): TypeD;
  abstract protected function transformation4(TypeD $input): TypeE;

  public function giveMeSomeThingInteresting(TypeA $input): TypeE {
    return $this->transformation4(
      $this->transformation3(
        $this->transformation2(
          $this->transformation1($input)
        )
      )
    );
  }
}
```

Method `giveMeSomeThingInteresting` returns a value which is a result of nested function calls.

In a given example it's quite readable becaue functions `transformation1`, `transformation2`, `transformation3`, `transformation4` take only one parameter, but it's easy to imagine that every one of them take some other additional arguments and this whole tree of nested calls get quite messy.

What you would do in such situation in languages like Haskell, Erlang/Elixir or F# you'd most probably use a pipeline operator ([Elixir example](http://elixir-lang.org/getting-started/enumerables-and-streams.html#the-pipe-operator)). If ther would be an easy way to pipe operations in PHP example above would change into:

```
class ALotOfNestedFunctionCallsExample {

  //it doesn't matter what those functions do internally
  abstract protected function transformation1(TypeA $input): TypeB;
  abstract protected function transformation2(TypeB $input): TypeC;
  abstract protected function transformation3(TypeC $input): TypeD;
  abstract protected function transformation4(TypeD $input): TypeE;

  public function giveMeSomeThingInteresting(TypeA $input): TypeE {
    return $this->operation1($input)
      |> $this->operation2($$)
      |> $this->operation3($$)
      |> $this->operation4($$);
  }
}
```

But PHP does not support such construct at the moment.

Such functionality was proposed in [PHP RFC: Pipe Operator](https://wiki.php.net/rfc/pipe-operator) at 2016-04-29, but it's not yet implemented.

So how can you get over it at the moment?

Use `Option` in a non-typical way.

```
class ALotOfNestedFunctionCallsExample {

  //it doesn't matter what those functions do internally
  abstract protected function transformation1(TypeA $input): TypeB;
  abstract protected function transformation2(TypeB $input): TypeC;
  abstract protected function transformation3(TypeC $input): TypeD;
  abstract protected function transformation4(TypeD $input): TypeE;

  public function giveMeSomeThingInteresting(TypeA $input): TypeE {
    return (new Some($this->operation1($input))
      ->map(function(TypeB $inp): TypeC {
        return $this->operation2($inp);
      })
      ->map(function(TypeC $inp): TypeD {
        return $this->operation3($inp);
      })
      ->map(function(TypeD $inp): TypeE {
        return $this->operation4($inp);
      })
      ->get();
  }
}
```

And with [extractors](../../01_Usage/03_Extractors.md) this can be squeezed to:

```
class ALotOfNestedFunctionCallsExample {

  //it doesn't matter what those functions do internally
  abstract protected function transformation1(TypeA $input): TypeB;
  abstract protected function transformation2(TypeB $input): TypeC;
  abstract protected function transformation3(TypeC $input): TypeD;
  abstract protected function transformation4(TypeD $input): TypeE;

  public function giveMeSomeThingInteresting(TypeA $input): TypeE {
    return (new Some($this->operation1($input))
      ->map(E($this)->operation2())
      ->map(E($this)->operation3())
      ->map(E($this)->operation4())
      ->get();
  }
}
```