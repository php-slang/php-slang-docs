### Using call_user_func
Using [call_user_func_array] is not recommended, but sometimes you may think it's impossible to live without it.

One example may be a method with unspecified count of parameters. Lets consider a following example.
```
function someFunc() {
  $args = func_get_args();
  // do something
}

call_user_func_array('someFunc',array('one','two','three'));
```

1st - it's a bad idea to use `func_get_args` as well.

2nd - example above can be transformed to:

```
function someFunc() {
  $args = func_get_args();
  // do something
}

someFunc(...['one','two','three']);
```
Or even better to:
```
function someFunc(...$args) {
  // do something
}

someFunc('one','two','three');
```

Shorter, more readable and less bug prone.