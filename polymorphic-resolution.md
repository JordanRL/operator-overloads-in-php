```php
class Number {
  function __construct(protected int|float $value) {}

  function __plus(Number $other, OperandPosition $position): Number
  {
    return new Number($this->value + $other->value);
  }
}

class Fraction extends Number {
  function __construct(protected int $numerator, protected int $denominator) {}

  function __plus(Number $other, OperandPosition $position): Number
  {
    if ($other instanceof Fraction) {
      $newNumerator = ($other->numerator * $this->denominator) + ($this->numerator * $other->denominator);
      $newDenominator = $this->denominator * $other->denominator;
    } else {
      $newNumerator = $this->numerator + ($other->value * $this->denominator);
      $newDenominator = $this->denominator;
    }
    
    $result = new Fraction($newNumerator, $newDenominator);
    $result = $result->reduce();
    
    return $result;
  }
}
```

## Without Polymorphic Handler Resolution

```php

$number = new Number(5);
$fraction = new Fraction(5, 8);

// All operators evaluated left-to-right

$result = $number + $fraction; // $result is Number(5), since the null property $value on the fraction gets cast to 0
$result = $fraction + $number; // $result is Fraction(45, 8), since the fraction implementation knows how to convert its parent class

```

## With Polymorphic Handler Resolution

```php

$number = new Number(5);
$fraction = new Fraction(5, 8);

// All operators evaluated left-to-right
// Unless the right-hand operator is a subclass of the left-hand operator

$result = $number + $fraction; // $result is Fraction(45, 8), since the fraction implementation knows how to convert its parent class
$result = $fraction + $number; // $result is Fraction(45, 8), since the fraction implementation knows how to convert its parent class

```
