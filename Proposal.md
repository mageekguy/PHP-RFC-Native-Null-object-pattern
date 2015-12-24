Hi internals@,

PHP support object oriented programming (aka OOP), and one of the base of OOP is polymorphism.
So it’s currently possible to use polymorphism in PHP code.
However, in some case, using it can be a PITA, because PHP does not implements natively the Null Object Pattern.
This is a (very) simple use case to illustrate that:

```
interface logger
{
	function messageIs($error);
}

class parser
{
	private $logger;

	function __construct(logger $logger)
	{
		$this->logger = $logger;
	}

	function parseData(parser\data $data)
	{
		…
		# Error, log it!
		$this->logger->messageIs($error);
		…
	}
}
```

Now, imagine that you don’t want to log any error.
One solution is to add a default value `null` to its argument:

```
class parser
{
	…
	function __construct(logger $logger = null)
	{
		$this->logger = $logger;
	}
	…
}
```

However, this (very) basic solution has a drawback: the developer must now check `$this->logger` everywhere before using it:

```
class parser
{
	…
	function dataProviderIs(data\provider $dataProvider)
	{
		…
		# Error, log it!
		if ($this->logger)
		{
			$this->logger->messageIs($error);
		}
		…
	}
}
```

A more elegant solution to avoid `if` everywhere is to use the null object pattern:

```
class blackholeLogger implements logger
{
	function messageIs($serror) {}
}

class parser
{
	…
	function __construct(logger $logger = null)
	{
		$this->logger = $logger ?: new blackholeLogger;
	}
	…
}
```

It’s a very better solution than the previous one because code complexity is not increased by `if` statement, and code readability is improved.
But now imagine that you use it in lot of classes: you must define lot of `blackhole*` classes, and doing that can be rapidly a PITA!
So, i think that it must be interesting to implements natively in the language the null object pattern as a class which has the following behavior:

1) an instance can replace any interface or class in term of type hinting (so the class implements or extends virtually all classes and interfaces);
2) all call to a method upon an instance return `$this` to avoiding the special case of returning "no object ».
3) an instance do absolutely nothing!

Its name can be:

- `blackhole`;
- `nullObject`;
- `UndefinedObject`;
- `nil` (as in Smaltalk, Lisp, …);
- or any other suggestion…

A pseudo coded implementation can be:

```
class nil extends * implements *
{
	function __call($method, $arguments)
	{
		return $this;
	}
}
```

With a native implementation of null pattern using `nil`, the `blackholeLogger` class become useless ans the previous code will be:

```
class parser
{
	…
	function __construct(logger $logger = null)
	{
		$this->logger = $logger ?: new nil;
	}
	…
}
```

In our opinion, the only drawback to introduce the null object pattern natively in PHP is backward compatibility, because the name of the class will become a reserved word, so maybe it can be implemented only in the future major version, aka PHP 8.

Any feedback?
