# Litter

**Litter is a pretty printer library for Go data structures to aid in debugging and testing.**

It's named for the fact that it outputs *literals*, which you *litter* your output with. Aa side benefit, all Litter output is compilable Go. You can use Litter to emit data during debug, and it's also really nice for "snapshot data" in unit tests, since it produces consistent, sorted output.

Litter was inspired by [Spew](https://github.com/davecgh/go-spew), but focuses on terseness and readability.

### Basic example

This:

```go
type Person struct {
  Name   string
  Age    int
  Parent *Person
}

litter.Dump(Person{
  Name:   "Bob",
  Age:    20,
  Parent: &Person{
    Name: "Jane",
    Age:  50,
  },
})
```

will output:

```
Person{
  Name: "Bob",
  Age: 20,
  Parent: &Person{
    Name: "Jane",
    Age: 50,
  },
}
```

### Use in tests

Litter is a great alternative to JSON or YAML for providing "snapshots" or example data. For example:

```go
func TestSearch(t *testing.T) {
  result := DoSearch()

  actual := litterOpts.Sdump(result)
  expected, err := ioutil.ReadFile("testdata.txt")
  if err != nil {
    // First run, write test data since it doesn't exist
		if !os.IsNotExist(err) {
      t.Error(err)
    }
    ioutil.Write("testdata.txt", actual, 0644)
    actual = expected
  }
  if expected != actual {
    t.Errorf("Expected %s, got %s", expected, actual)
  }
}
```

The first run will use Litter to write the data to `testdata.txt`. On subsequent runs, the test will compare the data. Since Litter always provides a consistent view of a value, you can compare the strings directly.

### Circular references

Litter detects circular references or aliasing, and will replace additional references to the same object with aliases. For example:

```go
type Circular struct {
  Self: *Circular
}

selfref := Circular{}
selfref.Self = &selfref

litter.Dump(selfref)
```

will output:

```
Circular { // p0
  Self: p0,
}
```

## Installation

```bash
$ go get -u github.com/sanity-io/litter
```

## Quick start

Add this import line to the file you're working in:

```go
import "github.com/sanity-io/litter"
```

To dump a variable with full newlines, indentation, type, and aliasing information, use `Dump` or `Sdump`:

```go
litter.Dump(myVar1)
str := litter.Sdump(myVar1)
```

### `litter.Dump(value, ...)`

Dumps the data structure to STDOUT.

### `litter.Sdump(value, ...)`

Returns the dump as a string

## Configuration

You can configure litter globally by modifying the default `litter.Config`

```go
// Strip all package names from types
litter.Config.StripPackageNames = true

// Hide private struct fields from dumped structs
litter.Config.HidePrivateFields = true

// Sets a "home" package. The package name will be stripped from all its types
litter.Config.HomePackage = "mypackage"

// Sets separator used when multiple arguments are passed to Dump() or Sdump().
litter.Config.Separator = "\n"
```

### `litter.Options`

Allows you to configure a local configuration of litter to allow for proper compartmentalization of state at the expense of some comfort:

``` go
  sq := litter.Options {
    HidePrivateFields: true,
    HomePackage: "thispack",
    Separator: " ",
  }

  sq.Dump("dumped", "with", "local", "settings")
```

