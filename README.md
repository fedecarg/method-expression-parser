# Introduction

Most ORMs support the concept of dynamic finders. A dynamic finder looks like a normal method invocation, but the method itself doesn't exist, instead, it's generated dynamically and processed via another method at runtime. MethodExpressionParser is a PHP library for parsing method expressions. It's designed to quickly and easily parse method expressions and construct conditions based on attribute names and arguments.

**Method Expressions**

GORM, Grails ORM library, introduces the concept of dynamic method expressions. A method expression is made up of the prefix such as "findBy" followed by an expression that combines one or more properties. Grails takes advantage of Groovy features to provide dynamic methods:

```
findByTitle("Example")
findByTitleLike("Exa%")
```

Method expressions can also use a boolean operator to combine two criteria:

```
findAllByTitleLikeAndDateGreaterThan("Exampl%", '2010-03-23')
```

In this case we are using AND in the middle of the query to make sure both conditions are satisfied, but you could equally use OR:

```
findAllByTitleLikeOrDateGreaterThan("Exampl%", '2010-03-23')
```

## Parsing Method Expressions

**Description**

```
[finderMethod]([attribute][expression][logicalOperator])?[attribute][expression]
```

**Expressions**

* LessThan: Less than the given value
* LessThanEquals: Less than or equal a give value
* GreaterThan: Greater than a given value
* GreaterThanEquals: Greater than or equal a given value
* Like: Equivalent to a SQL like expression
* NotEqual: Negates equality
* IsNotNull: Not a null value (doesn't require an argument)
* IsNull: Is a null value (doesn't require an argument)

## Examples

```
findByTitleAndDate('Example', date('Y-m-d'));
SELECT * FROM book WHERE title = ? AND date = ?

findByTitleOrDate('Example', date('Y-m-d'))
SELECT * FROM book WHERE title = ? OR date = ?

findByPublisherOrTitleAndDate('Name', 'Example', date('Y-m-d'))
SELECT * FROM book WHERE publisher = ? OR (title = ? AND date = ?)

findByPublisherInAndTitle(array('Name1', 'Name2'), 'Example')
SELECT * FROM book WHERE publisher IN (?, ?) AND date = ?

findByTitleLikeAndDateNotNull('Examp%')
SELECT * FROM book WHERE title LIKE ? AND date NOT NULL

findByIdOrTitleAndDateNotNull(1, 'Example')
SELECT * FROM book WHERE (id = ?) OR (title = ? AND date NOT NULL)
```

### Example 1

```
findByTitleAndDate('Example', date('Y-m-d'));
```

Outputs:

```
array
  0 =>
    array
      0 =>
        array
          'attribute' => string 'title'
          'expression' => string 'Equals'
          'format' => string '%s = ?'
          'placeholders' => int 1
          'argument' => string 'Example'
      1 =>
        array
          'attribute' => string 'date'
          'expression' => string 'Equals'
          'format' => string '%s = ?'
          'placeholders' => int 1
          'argument' => string '2010-03-22'
```

### Example 2

```
findByTitleOrDate('Example', date('Y-m-d'));
```

Outputs:

```
array
  0 =>
    array
      0 =>
        array
          'attribute' => string 'title'
          'expression' => string 'Equals'
          'format' => string '%s = ?'
          'placeholders' => int 1
          'argument' => string 'Example'
  1 =>
    array
      0 =>
        array
          'attribute' => string 'date'
          'expression' => string 'Equals'
          'format' => string '%s = ?'
          'placeholders' => int 1
          'argument' => string '2010-03-22'
```

### Example 3

```
findByPublisherOrTitleAndDate('Name', 'Example', date('Y-m-d'));
```

Outputs:

```
array
  0 =>
    array
      0 =>
        array
          'attribute' => string 'publisher'
          'expression' => string 'Equals'
          'format' => string '%s = ?'
          'placeholders' => int 1
          'argument' => string 'Name'
  1 =>
    array
      0 =>
        array
          'attribute' => string 'title'
          'expression' => string 'Equals'
          'format' => string '%s = ?'
          'placeholders' => int 1
          'argument' => string 'Example'
      1 =>
        array
          'attribute' => string 'date'
          'expression' => string 'Equals'
          'format' => string '%s = ?'
          'placeholders' => int 1
          'argument' => string '2010-03-22'
```

### Example 4

```
findByPublisherInAndTitle(array('Name1', 'Name2'), 'Example');
```

Outputs:

```
array
  0 =>
    array
      0 =>
        array
          'attribute' => string 'publisher'
          'expression' => string 'In'
          'format' => string '%s IN (%s)'
          'placeholders' => int 1
          'argument' =>
            array
              0 => Name1
              1 => Name2
      1 =>
        array
          'attribute' => string 'title'
          'expression' => string 'Equals'
          'format' => string '%s = ?'
          'placeholders' => int 1
          'argument' => string 'Example'
```

### Example 5

```
findByTitleLikeAndDateNotNull('Examp%');
```

Outputs:

```
array
  0 =>
    array
      0 =>
        array
          'attribute' => string 'title'
          'expression' => string 'Like'
          'format' => string '%s LIKE ?'
          'placeholders' => int 1
          'argument' => string 'Examp%'
      1 =>
        array
          'attribute' => string 'date'
          'expression' => string 'NotNull'
          'format' => string '%s IS NOT NULL'
          'placeholders' => int 0
          'argument' => null
```

### Example 6

```
findByTitleAndPublisherNameOrTitleAndPublisherName('Title', 'Name1', 'Title', 'Name2');
```

Outputs:

```
array
  0 =>
    array
      0 =>
        array
          'attribute' => string 'title'
          'expression' => string 'Equals'
          'format' => string '%s = ?'
          'placeholders' => int 1
          'argument' => string 'Example'
      1 =>
        array
          'attribute' => string 'publisher_name'
          'expression' => string 'Equals'
          'format' => string '%s = ?'
          'placeholders' => int 1
          'argument' => string 'Name1'
  1 =>
    array
      0 =>
        array
          'attribute' => string 'title'
          'expression' => string 'Equals'
          'format' => string '%s = ?'
          'placeholders' => int 1
          'argument' => string 'Example'
      1 =>
        array
          'attribute' => string 'publisher_name'
          'expression' => string 'Equals'
          'format' => string '%s = ?'
          'placeholders' => int 1
          'argument' => string 'Name2'
```

## Usage

```
class ActiveRecord
{
    private $methodExpressionParser;

    public function getMethodExpressionParser() {
        if (! isset($this->methodExpressionParser)) {
            $this->methodExpressionParser = new MethodExpressionParser();
        }
        return $this->methodExpressionParser;
    }

    public function findBy($conditions) {
        var_dump($conditions);
    }
    public function findAllBy($conditions) {
        var_dump($conditions);
    }

    // Invoke finder methods
    public function __call($method, $args) {
        if ('f' === $method{0}) {
            try {
                $result = $this->getMethodExpressionParser()->parse($method, $args);
                $finderMethod = key($result);
                $conditions = $result[$finderMethod];
            } catch (MethodExpressionParserException $e) {
                $message = sprintf('%s: %s()', $e->getMessage(), $method);
                throw new Exception($message);
            }
            return $this->$finderMethod($conditions);
        }

        $message = 'Invalid method call: ' . __METHOD__;
        throw new BadMethodCallException($message);
    }
}
```

## Performance

PHP doesn't allow you to define methods dynamically, this means that every time you invoke a finder method the parser has to search, extract and map all the attribute names and expressions. To avoid introducing this performance overhead you can cache the attribute names. For example:

```
class ActiveRecord
{
    private $methodExpressionParser;
    private $classMetadata;

    public function getMethodExpressionParser() {
        //...
    }

    public function getClassMetadata() {
        //...
    }

    // Invoke finder methods
    public function __call($method, $args) {
        if ('f' === $method{0}) {
            $parser = $this->getMethodExpressionParser();
            $classMetadata = $this->getClassMetadata();
            try {
                $finderMethod = $parser->determineFinderMethod($method);
                if ($classMetadata->hasMissingMethod($method)) {
                    $attributes = $classMetadata->getMethodAttributes($method);
                    $conditions = $parser->map($args, $attributes);
                } else {
                    $expressions = substr($method, strlen($finderMethod));
                    $attributes = $this->extractAttributeNames($expressions);
                    $conditions = $parser->map($args, $attributes);
                    $classMetadata->setMethodAttributes($method, $attributes);
                }
            } catch (MethodExpressionParserException $e) {
                $message = sprintf('%s: %s()', $e->getMessage(), $method);
                throw new Exception($message);
            }
            return $this->$finderMethod($conditions);
        }

        $message = 'Invalid method call: ' . __METHOD__;
        throw new BadMethodCallException($message);
    }
}
```

The Expression objects are lazy-loaded, depending on the expressions found in the method name.

## Extensibility

The MethodExpressionParser class was designed with extensibility in mind, allowing you to add new Expressions to the library.

```
abstract class Expression {
}
class EqualsExpression extends Expression {
}
```

## License

* Copyright (c) 2010, Federico Cargnelutti. All rights reserved. 
* New BSD License http://www.opensource.org/licenses/bsd-license.php
