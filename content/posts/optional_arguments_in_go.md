+++
title = 'Optional arguments in Go'
date = 2024-07-21T17:10:09+08:00
draft = true
+++


## Zero Values

```golang
func greet(name string, age int) {
    if name == "" {
        name = "Guest"
    }
    if age == 0 {
        age = 18
    }
    fmt.Printf("Hello, %s! You are %d years old.\n", name, age)
}
```


## Structs as Parameters

```golang
type GreetOptions struct {
    Name string
    Age  int
}

func greet(opts GreetOptions) {
    if opts.Name == "" {
        opts.Name = "Guest"
    }
    if opts.Age == 0 {
        opts.Age = 18
    }
    fmt.Printf("Hello, %s! You are %d years old.\n", opts.Name, opts.Age)
}
```


## Variadic Functions


```golang
func (s *Some) Close(arg ..context.Context) {
    ctx := context.Background()
    if len(arg) > 0 {
        ctx = arg[0]
    } 
    
    // Closing logic
}
```


## Functional Options

```golang
type GreetOptions struct {
    Name string
    Age  int
}

type Option func(*GreetOptions)

func WithName(name string) Option {
    return func(opts *GreetOptions) {
        opts.Name = name
    }
}

func WithAge(age int) Option {
    return func(opts *GreetOptions) {
        opts.Age = age
    }
}

func greet(options ...Option) {
    opts := GreetOptions{
        Name: "Guest",
        Age:  18,
    }
    for _, opt := range options {
        opt(&opts)
    }
    fmt.Printf("Hello, %s! You are %d years old.\n", opts.Name, opts.Age)
}
```


## Builder

```golang
type GreetBuilder struct {
    name string
    age  int
}

func NewGreetBuilder() *GreetBuilder {
    return &GreetBuilder{
        name: "Guest",
        age:  18,
    }
}

func (b *GreetBuilder) WithName(name string) *GreetBuilder {
    b.name = name
    return b
}

func (b *GreetBuilder) WithAge(age int) *GreetBuilder {
    b.age = age
    return b
}

func (b *GreetBuilder) Greet() {
    fmt.Printf("Hello, %s! You are %d years old.\n", b.name, b.age)
}
```

## Method Overloading (Simulated)

```golang
// Simulated method overloading by creating multiple functions
func greetWithName(name string) {
    greet(name, 18)
}

func greetWithAge(age int) {
    greet("Guest", age)
}

func greetWithNameAndAge(name string, age int) {
    greet(name, age)
}

func greet(name string, age int) {
    fmt.Printf("Hello, %s! You are %d years old.\n", name, age)
}
```

## Maps for Dynamic Parameters

```golang
func greet(params map[string]string) {
    name := "Guest"
    age := 18

    if val, ok := params["name"].(string); ok {
        name = val
    }
    if val, ok := params["age"].(int); ok {
        age = val
    }

    fmt.Printf("Hello, %s! You are %d years old.\n", name, age)
}

// Usage
params := map[string]interface{}{
    "name": "Alice",
    "age":  30,
}
greet(params)
```
