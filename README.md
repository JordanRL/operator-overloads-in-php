# Introduction

Operator overloads are a feature that will be new to many PHP developers. As such, I've created this document as a way for PHP developers to start learning *good* habits with the feature. **This is not a substitute for the actual PHP.net documentation.**

**NOTE:** This feature has not been accepted as an RFC yet.

# Rule 1: All Overloads Must Be Immutable

It is critically important in the vast majority of programs that objects which use operator overloads implement them immutably. For those who are not familiar, an object is *immutable* if it prevents changes to its values after some initialization period. Typically, this is the constructor method. However, individual methods can treat the object as immutable. Consider the following two examples.

## Example 1 (Mutable Implementation)

```php
<?php

class SomeClass {
  protected int $somevar = 0;
  
  public function increment(): SomeClass {
    $this->somevar++;
    
    return $this;
  }
}
```

## Example 2 (Immutable Implementation)

```php
<?php

class SomeClass {
  public function __construct(protected int $somevar) {}
  
  public function increment(): SomeClass {
    $newObj = new SomeClass($this->somevar + 1);
    
    return $newObj;
  }
}
```

These two examples show the key thing to understand about a mutable vs. an immutable implementation: a mutable implementation *modifies the existing object* while an immutable implementation *returns a new object that has the modifications already performed on it*.

So why is this so important for operator overloads? *Because programmers expect operators to do things immutably ALWAYS.*

Consider what we expect the value of `$x` to be throughout this code:

```php
<?php

$x = 0; // We expect $x to be 0

$x = $x + 2; // We expect $x to be 2

$y = $x + 5; // We expect $x to be 2

$x += 3; // We expect $x to be 5

++$x; // We expect $x to be 6

$z = $y + 3; // We expect $x to be 6
```

Now consider what would *actually* happen if $x behaved like a mutable object:

```php
<?php

$x = 0; // $x == 0

$x = $x + 2; // $x == 2

$y = $x + 5; // $x == 7, $y == 7 (is a reference to $x)

$x += 3; // $x == 10

++$x; // $x == 11

$z = $y + 3; // $x == 14, $y == 14, $z == 14 ($y and $z are references to $x)
```

So this is the first rule of operator overloads: you cannot implement them immutably. That means that you cannot have *any* lines in the operator method that look like `$this->property = 'somevalue'`.

The exceptions to this rule should be extremely rare, and should require a great deal of care. If you think you have an exception, you probably don't. Don't violate this rule, people will hate your code for it.

# Rule 2: Don't Be Too Clever

It really, really sucks when the *problem* with code is not the same place that the *error* happens. If you try to make your operator overload *too* clever so that it can handle all sorts of edge cases, you're probably doing it wrong. Instead, you want your operator overload implementations to be *very* picky about the input data and types, and to error if anything seems out of place.

Consider the following:

```php
<?php

class Money {
  public function __construct(readonly public float $value, readonly public string $currency) {}
  
  public function getConvertedValue($currency): float {
    // Perform currency conversion
  }
  
  operator +(Money $other, OperandPosition $operandPos) {
    if ($other->currency != $this->currency) {
      if ($operandPos == OperandPosition::LeftSide) {
        $newCurrency = $this->currency;
        $newValue = $this->value + $other->getConvertedValue($newCurrency);
      } else {
        $newCurrency = $other->currency;
        $newValue = $this->getCovertedValue($newCurrency) + $other->value;
      }
    } else {
      $newCurrency = $this->currency;
      $newValue = $this->value + $other->value;
    }
    
    return new Money($newValue, $newCurrency);
  }
}
```

So what exactly is going on here? Well, this implementation is trying to automatically convert currencies if they don't match. Could this sometimes be the correct thing to do? Yes. But most of the time, it would probably be clearer to force the currency conversion to be explicit in the calling code. This implementation probably won't produce any bugs on its own, but what if the currency conversion is based on an API?

If the API breaks, we could see errors in code paths that don't appear to ever call the currency conversion code at all! All we see is us adding two money values. Worse, the error wouldn't indicate what line in our program actually caused the problem without a full backtrace or using xDebug on production! If we mock the API endpoint in our dev environment, we may not even be able to reproduce the error at all.

How clever your code should be is not something there is a single answer to. For instance, a library that does unit values with automatic conversions may have the *expectation* that it would automatically handle such things with operator calls. But if the operator does some kind of work that wouldn't typically be expected by someone using the operator in code, it's probably being too clever.

# Rule 3: Don't Allow Ambiguity

Continuing with the money example we used in the last section, what if we allowed the operator to accept both `int` and `float` also? Seems like a great idea.

```php
<?php

class Money {
  public function __construct(readonly public float $value, readonly public string $currency) {}
  
  public function getConvertedValue($currency): float {
    // Perform currency conversion
  }
  
  operator +(Money|int|float $other, OperandPosition $operandPos) {
    if ($other instanceof Money) {
      if ($other->currency != $this->currency) {
        throw new Exception('Currencies must match');
      }
      $otherValue = $other->value;
    } else {
      $otherValue = $other;
    }
    
    $newValue = $this->value + $otherValue;
    
    return new Money($newValue, $this->currency);
  }
}
```

Great! We even fixed the problem of different currencies from Rule 2. Or... did we?

```php
<?php
$money = new Money($_GET['amount'], $_GET['currency']);

$money += 5; // $5 fee added

...
```

It seems like this should work, except... it *won't* add $5, it'll add 5 in whatever currency `$money` happens to be in! If it's *in* dollars then it will work as intended, but what if it's in Euros, or Pounds, or Yuan, or Yen? Adding 5 Yen instead of 5 dollars would mean that our result is off by quite a lot! $1 US is about 100 Yen, so in that case we should be adding nearly 500 instead.

So couldn't we add a conversion factor for ints and floats? We could. It would have some of the same problems explored in Rule 2, but it's possible. However, that assumes that all developers are programming in the same currency as you. What happens if one of your colleagues thinks in a different currency? What if they get assigned a feature to add a 25 Ruble surcharge, and don't realize that the program assumes all integers are in dollars?

In this case, integers and floats are missing an important dimension: the currency. It's unsafe to assume the intended currency of a particular integer or float. That's in fact the whole *point* of the Money class, to supplement the integer and float types because they don't capture this information.

# Rule 4: Supporting Implied Operators Is Non-Optional

The reassignment operators, such as `+=` and `*=` are referred to as "implied operators" in the RFC. The reason for this is that these operators are optimized within the PHP engine separate from this RFC.

```php
<?php

$x += 5; // My code contains the reassignment operator
```

However, the PHP engine actually executes this line as:

```php
<?php

$x = $x + 5; // The operation that is executed
```

The two lines are not 100% equivalent, as a different part of the VM is run for both of the lines above. The difference is that for the `$x += 5` line, the engine *first* executes logic that is specific to reassignment operators (such as allocating temporary values) and *then* calls the normal opcode for the `+` operator.

This is also the case for post-increment and pre-increment, `++$x` and `$x++`. In these lines, the engine *first* executes code that is specific to the increment and *then* calls the normal operator. For the post-increment line, it sets the return value of the line *before* calling the normal operator, while for the pre-increment line it sets it after.

All of the operators that are listed under "implied operators" are non-optional. Not only is there no way to overload the normal operators *without* overloading the coresponding implied operators, but doing so would involve removing or significantly altering many optimizations within the PHP engine that are unrelated to operator overloads.

Because of this, you must design your operator overloads with the understanding that you cannot prevent the implied operators from being supported if you support the associated normal operator.

**NOTE**: By not accepting integers in the operator overload, you can cause the `++$x` and `$x++` lines to result in an error. 

# Rule 5: Operands Are Never Passed By Reference

Lets look at a usage of the operator overloads that would probably be contraversial among many PHP developers:

```php
<?php

$item = new Item();
$queue = new Queue();

$queue = $queue << $item;
```

This code attempts to use the bitwise shift left operator to put an item into a queue. If you did something like this, it would follow that you could use the bitwise shift right operator to pull the next item out of the queue.

```php
<?php

$queue = $queue >> $nextItem;
```

There are several problems with this usage of operator overloads. First, the operator's meaning is reversed if the queue is on the right instead of the left: `$queue >> $item` vs. `$item >> $queue`. Second, you cannot pass the `$other` parameter of the operator overload by-reference, making it difficult to assign values to the variable. Third, it is very difficult to implement this behavior without violating Rule 1.

In general, if you need to operate on the *reference* of the other operand, you are probably using operator overloads incorrectly as designed in this RFC.
