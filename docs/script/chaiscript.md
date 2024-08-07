# ChaiScript Overview

ChaiScript is an embedded scripting language designed for C++. You can use ChaiScript to expand the functionality of RealmJoin. Use the following script examples and the scripts in the [GK Script Objects](reference/) section to learn how to customize your RealMigrator.

To download ChaiScript click the following link: [http://chaiscript.com](http://chaiscript.com)

## Variables

```c++
var i; // uninitialized variable, can take any value on first assignment;

var k = 5; // initialized to 5 (integer)
var &m = k; // reference to k
```

## Built in Types

```c++
var i = 1; // creates an integer
var u = 1u; // creates an unsigned integer
var s = "hello"; // create a string
var f = 1.0; // creates a float

var v = [1,2,3u,4ll,"16", `+`]; // creates vector of heterogenous values
var m = ["a":1, "b":2]; // map of string:value pairs
```

### Working with strings

```shell
chaiCmd.exe
eval> var s="Hello";            // Assignment
Hello
eval> s+= " World !!";          // Concatenation
Hello World !!
eval> s.size();                 // Size
14
eval> s.find("World");          // Find
6
eval> s[6];                     // reference character
W
eval> s.substr(0, 6);           // Substrings
Hello
eval> s.substr(6,-1);
World !!
eval> s.find("o");              // Find - reverse find
4
eval> s.rfind("o");
7
```

## Conditionals

```c++
if (expression) { }

if (expression) { }
else { }

```

## Loops

```c++
// c-style for loops
for (var i = 0; i < 100; ++i) { print(i); }
```

```c++
// while
while (some_condition()) { /* do something */ }
```

```c++
// ranged for
for (x : [1,2,3]) { print(x); }
```

Each of the loop styles can be broken using the break statement. For example:

```
while (some_condition()) {
  /* do something */
  if (another_condition()) { break; }
}
```

## Functions

```c++
def myfun(x, y) { x + y; } // last statement in body is the return value
def myfun(x, y) { return x + y; } // equiv
```

### Optionally Typed

```c++
def myfun(x, int y) { x + y; } // requires y to be an int
```

## Classes/Objects

### Class definition

```c++
class MyType {
  var value;

  def MyType() {
    this.value = "a";
  }

  def get_value() {
    "Value Is: " + this.value;
  }
};
```

### Working with class objects

```c++
var m = MyType(); // calls constructor
print(m.get_value()); // prints "Value Is: a"
m.value += " joke";
print(m.get_value()); // prints "Value Is: a joke"
```

## JSON

| Script      | Function                                                                                                                           |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `from_json` | Converts a JSON string into its strongly typed (map, vector, int, double, string) representations                                  |
| `to_json`   | Converts a ChaiScript object (either a object or one of map, vector, int, double, string) tree into its JSON string representation |
