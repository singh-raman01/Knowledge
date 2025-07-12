Recommended Book: The GO programming language, by Alan A Donovan
### Chapter 1: why GoLang?
Tl;dr: *Simplicity*

### Chapter 2: Benefits

*The go.mod file*: We can create any directory, with any module name.
And then initialise `go mod init` inside it.
This will contain all the dependencies (even 3'rd party inside it)

### Chapter 3: Types 

![[Pasted image 20250629104438.png]]
Keywords and Operators in GoLang: Tl;dr: *very few*

`a := 2` Then a is an address for the memory location for that number

 *How does variable assignment work?*
Basically you have 3 simple ways,

```go

// First 2 can be done anywhere
var a int

var (
  b = 2
  f = 2.01
)

// This can only be done inside the functions
c := 2 // := ka the short declaration operator

b = int(f)


// we use var when we don't want to initalise
// or we use the short hand declaration when we want to intialise

// Float can be changed to int, but will lose information
// if converted back into float
```

*%x knowledge*
```c
a := 2
fmt.Printf("%8T %[1]v\n",a,a)
// T shows the type
// v just shows the value, and the value can be anything
// [1] means that for the second argument, you can just use the first variable
```

*bool* : cannot be converted from/to ints, in go, bool is totally different from int

*error type* : a special type which has one function Error(), and either it's nil or it's not

*pointer* : either nil or not (nil means not appointed to anything)

*Initialisation* : in go all variables are initialised as soon as declared, by us or by go

*constants* : only numbers, string and booleans. 
	Why not objects?
	 Because go is a complied time language, and objects can be changed, they are not immutable (this is not safe in concurrent programming)

  *if statements* : can be done in a single or a multi-line

### Chapter 4 : String 

Unicode is a number that is bigger than bytes, which is used to represent characters in string.

`byte` is unit8
`rune` is int32

Physically characters and an encoding of runes into UTF-8 characters.
UTF-8 allows to store and transmit unicode text. (Surpassed ASCII which only has 128 characters)
UTF-8 uses 1-4 bytes for each character.
![[Screenshot 2025-06-29 at 11.25.02 am.png]]

*What is the length of the string*? 
We get the physical answer: i.e. 6 bytes
	Length of string is the number of bytes it takes to encode it into UTF and not the runes.(unicode characters)

`s := Hello world`
`hello := s[:5]`
`world := [7:]`
*Then what is s* ?
 s is a descriptor of the string, in our case 
 ```go 
 s = {
 //data Points to the first char of string
 len = 12}
 
 hello = {
 //data points to w
 len 5}

x,y := y,x // works to swap the values
```
Thus there is no concept of the ending character in golang.

*string are immutable and can share the underlying storage*
meaning that hello and s can share the same string data memory.

*overloading* string can overload the `+` or `+=` character.

But if the strings are immutable, how does += work?
Solution: *copy*
The same way you have functions like: `s = strings.ToUpper(s)`


*The comma-OK idiom in golang* : `value, ok := someMap[key]` 
The map returns the value and the bool if the value was present or not. So in case the value is not present, you have value as nil or empty depending on the type and the ok is a bool.
It's a built in operation to ask for the value and a bool asking if it really existed or not.

### Arrays, Slices and Maps

``` go
// Declarations in go

[5]int : //array
[...]{1,2,4} : //means the no of elements I put in the array will be my size of the array

[]int : //slice
[]int{0,0,0}: //slice

map[string]int : //map
```

- Arrays have fixed size
- Arrays are passed by values, hence the elements are copied
- sized by initialiser (thus, cannot copy array of different sizes)

- Slices are passed by reference, and no copying, referencing is OK
- `a = append(a,1)` copy and allocate a bigger chunk of memory to A
- `e := a` then changing e also changes a
- [8:11] means elements from 8 to 10
- Slices are not comparable, but arrays are
- Slices cannot be used as map key, but arrays can be used as map keys
- We cannot copy an array into a slice and a slice into an array, we can do so, but it won't be a complete copy



- The key in a Map has to be comparable
- You can read from a `nil` map but `insert` will panic.
- Maps are dictionaries
```go 
var m map[int]string // here the map is not initialized
// and does not point to any hash table
p := make(map[int]string) // here we initialize the map

SO,
m is nil
p is not nil but empty

if we read a key that's not there, we get 0. (or whatever is the default value for that type)

m = p \\ works like the string assignment. Then m is the same as p now because they point to the same hash table now

The type that you use as the key in map should have the 
!= and == operators defined in it

map cannot be compared to another map
Cannot ask the capacity of map

There are 2 ways to index a map:

a := p["the"]    // 0
a,OK := p["the"] //0,false

Thus the 2'd way is good to have be used in the if statements
```

![[Pasted image 20250706101519.png]]

Nil are a type of zero, and functions like `len, cap, range` can make use of nil.
Example: when a slice has no elements `var s []int` means that s is nil.
So when we try to `l := len(s)` we get 0.

### Chapter 6: Control statements, declarations and types

- All if-else statements require braces in go
-  if else can start with a short declaration, in the example below we declared the variable err and then used it 
```go
if err := doSomething(); err!=nil{

}
```

- In the for loops, all three parts are optional (initialize, check, increment)
```go
for i := 0; i <10; i++ {

} 
```
- loops fail when explicit check fails

```go
for i := range dataObject {
 
}

for i, v := range dataObject{

}
```

- In the first case, we get the index of the data object (in case of the map, it is the key)
- In the second case, we get the index and the value (the value is copied)
- In case of the map, range doesn't get the same order of keys, we need an ordered map for this
```go
for {

// break statement
}

for an infinite loop

for _,v := {
}

then you would get the value, and the index gets ignored

```


Sometimes, we need to break or continue the outer loop (maybe a nested loop for quadratic research)
Then you need to add a label to your loop statement

```go
outer:
	for k := range x {
		for y := range z {
				if someCondition {
					continue outer
			}
			}
	}
```

Switch is an alternate to the if-else (syntactic sugar)
```go
switch a := f.Get(); a {
	case 0,1,2:
		doSomething()
	case 4,5,6:
		//normal to not have anything inside the switch
	default:
		doAnotherthing()
}
```
- One case doesn't fall through the next in Golang, to see fallthrough happens you have to do it explicitly.
- `Default` can be put anywhere, but will always be evaluated at last

 ```go
 a := someFunc()

	switch {
		case a <=2:
			x()
		case b <=6:
			y()
		default:
			//
	}
```

This works like normal if-else, i.e. fall through happens


*Packages*
- Everything in Golang lives inside some package.
- Nothing is global, it's either in the package scope or the function scope.
- Short Declarations can be used only inside the functions, we'll understand why
```go

package main

const x = "qkwndaknd"
var someKey string
type someStruct struct {
///
}

func doSomething() error{

}
```
As we see here, everything is declared with a keyword, because it's easier to parse (???)
- Every name that is capitalised is exported, otherwise it's private to the package

- Each source file in the package must import what it needs, otherwise you get an error
- We cannot have circular dependencies.
	- Solution: Move the common thing into another package.

- Items within a package get initialised before main
*What makes a good package?*
Only export the API that you want the package to show.


*Declarations*
- constants with const
- variables with var (must have type or initial value, or sometimes both)
- shorthand := (only inside the function
	- It must be used (not var, not const) inside the control statment
	- It must declare at least one new variable
	- It won't re-use an existing declaration from an outer scope
- functions with func
- Formal parameters and named returns of a functions ??

```go
err := doSomething()
err := doAnotherthing() // this is wromg
x, err := getSomeValue() // OK: err is not redeclared 

func main()
	n,err := fmt.Println("Hello world")

	if _,err := fmt.Println(n); err!=nil{
			do()
		}


// this function gives an error on the first err declaration because it was unused, as that err was declared on the main function scope, and the new error was declared on the if scope

	var x int
	for {
		x := 2
	}
	return x
	// this x is nil, because the one declared inside is from another scope 
```

For assignments, the size matters too!
- An array of size 4 and 5 cannot be assigned to one another 

*Named typing*

```go

type x int

func main(){
	var a x
	b := 12
	a = b // FAILS mismatch
	a = 12
	a = x(b) //OK 
}
```

*Operators*
- only + is overloaded (for string)
- comparators work with string and numbers
- Boolean operators(! && ||) only work with booleans
- Bitwise (& | ^ << >> &^) work on ints
- There is 5 levels of operators precedence


### Formatted and File I/O

- unix has the concept of three standard I/O streams
- They are open by default in every program
	- Standard Input
	- Standard Output
	- Standard Error
- They are mapped to the console/terminal but can be redirected For example to text files.

![[Screenshot 2025-07-06 at 10.14.17 pm.png]]

| Function   | Destination | Formatting | Newline | Use Case                     |
| ---------- | ----------- | ---------- | ------- | ---------------------------- |
| `Println`  | stdout      | No         | Yes     | Quick logging                |
| `Printf`   | stdout      | Yes        | No      | Precise formatting           |
| `Fprintln` | io.Writer   | No         | Yes     | Logging to file/stream       |
| `Fprintf`  | io.Writer   | Yes        | No      | Formatted file/socket output |
| `Sprintln` | string      | No         | Yes     | Build simple string          |
| `Sprintf`  | string      |        Yes | No      | Compose formatted string     |
We have different packages related to file I/O:
- Package `os` has functions to open and create files, list directories and hosts the `File` type
- Package `io` has utils to read and write; `bufio` provides buffered I/O scanners
- Package `io/ioutil` has extra utilities such as reading an entire file to memory or, writing it all out at once.
- Package `strconv` has utils to convert to/from string representations
- NOTE: `strconv` is the best way to get numbers back and forth from texts, because it is designed for it.

	NOTE: [[General Knowledge]] Understand why this is referenced here

*GO Convention*: Whenever an error is returned, always check for it. Check and handle that error

### Functions, Parameters and Defer

Functions are first class citizens,  so we can do all the things with functions as we do with for example strings or ints.
 - We can define them - even inside other functions
 - Create anonymous function literals
 - Pass them as returns/ parameters
 - Store them as variables
 - Store them in slices and maps (not as keys)
 - Store them as fields of a structure type
 - Send and receive them in channels
 - Write methods against a function type
 - Compare a function var against a nil
 - Almost anything can be defined inside a function(struct, var, another func)
 - Methods cannot be defined in a function 
 - The signature of a function is the order and type of it's parameters and return values
Things passed by value: // the values are copied, changes are local
- Numbers, bools, arrays, structs
Things passed by reference:
- things passed by pointers, strings, slices, maps, channels

```go

func do(m1 map[int]int) {
	m1[3]=0
	m1 = make(map[int]int)
	fmt.Println("m1",m1)
}

func main(){

	m := map[int]int{4:1,2:4,5:6}
	do(m)
	fmt.Println("m",m)
	
}

// the output is :
// m1 map[4:4]
// m map[4:1 2:4 3:0 5:6]
```
The above example shows that in maps you have pass by reference, and m1 is initially referenced to the variable m, but then we assign it another hash table using the `make` . Thus m1 is a local variable inside.
To make sure m is modified you need to do `*m1 = make(map[int]int`
and also you need to define the function as `func(m1 *map[int]int`)
*Conclusion*:
All parameters in hindsight, are passed by value.
Parameters are passed by copying something.
If the copied parameter is a pointer or a descriptor, then the shared backing store can be changed through it. (array, hash table)


- GO can have multiple return values.
- `func x(int) (int,err) {}`

*Defer*:
A defer statement captures a function call to run later
Defer operates on a function scope, and not the block scope.
What does this mean?
```go

func main(){

	f := os.Stdin

	if len(os.Args)>1 {
		
		if f,err := os.Open(os.Args[1]); err!=nil{
			
		}
		defer f.Close()
	}
	
}
```
- Here,  defer will execute if we enter the if statement, but will not if we do not enter, if the defer was block scoped, then it should run after the ending of the if statement but it runs when the function main() finally exits.
 - Thus, defer will happen only when the function exits.
 - The value of the function/anything inside the defer is copied.
	 `defer fmt.Println(a)` then a is a copied value.

 *Naked returns and defer*:
 ```go

func todo() (a int){
defer func(){
a = 2
}()

a = 1
return
}
```

here, we need not return anything, we use the return value variable a as a variable inside the function, and when the function finally exits we have a =2 because of the defer.

### Chapter 9: Closures

Scope of anything is static, based on the code at compile time.
But the lifetime depends on the program execution.(runtime)

```go
func do() *int{

return &b
}
```
Then the value(object) will live so long as part of the program keeps a pointer to it.
Even though the scope of variable b delcared inside the do() function has ended. Go has this inside implementation which handles the niche details.

Go has a bigger stash than heap and puts most of the things inside it, but if the variable has to be for lifetime then it's put into heap.

*What is a closure*?

Closures captures the variables, and not the reference.

```go

func fib() func() int {

	a,b := 0,1
	return func() int {
		a,b = b, a+b
		return b
	}
}
```

We notice that the inner function doesn't declare a & b but borrow them from the outer function. Thus their lifetime becomes more than expected.

It's a runtime thing, and we usually don't think about it. 
it's like a string descriptor.

![[Screenshot 2025-07-07 at 12.26.06 pm.png]]
On the left hand side f is just a function, but for the right hand side, f becomes a closure.

```go

func main(){
	f,g := fib(), fib()
	for x := f(); x< 100; x=f() {
		fmt.Println(x)
	}
}


```
Then as long as f exists, we can use the variables a & b
g has different variables (own copy of a & b).

When we look at an old example:

```go

var ss []kv

sort.Slice(ss, func(i, j) bool {
	return ss[i].val > ss[j].val
})
	
```


The sort.Slice took as input a slice and a closure.
This closure has a particular signature determined by the slice inputs.
we are using the `ss` directly inside the function without passing it.
The closure gets `ss` from the var declaration.

*When does closure come for need*?

When the function that you can create is fixed by type, so you need to borrow variables from outside the scope of your function.


### Slices in Details

```go

var a []int

b := []int{}

c := make([]int, 5)

d := make([]int, 0, 5)

// OUTPUT
//0, 0, []int,  true, []int(nil)
//0, 0, []int, false, []int{}
//5, 5, []int, false, []int{0, 0, 0, 0, 0}
//0, 5, []int, false, []int{}
```


![[Screenshot 2025-07-07 at 1.27.02 pm.png | 400]]
The left hand parts of the image show that for each slice variable you have:
 - length
 - capacity
 - Address

Then based on the way we initialise the object, we get different values in these fields.

*Why does knowing this matter to us*?

One example can be using json.
Json encoding works differently in JSON when the object is nil.

For `s` , json marshal will return `nil`
For `t`, json will return `[]`

In case of the variable u, when we do the append, it gets appended after all the zeroes.

Appending to a nil slice is okay, because the slice is copied and new memory is allocated, unlike inserting in a nil map. 

```go
a := []int{1, 2, 3}

b := a[0:1]

c := b[0:2]

  

fmt.Println("a= ", a)

fmt.Println("b= ", b)

fmt.Println("c= ", c)

  

fmt.Println(len(b))

fmt.Println(cap(b))

fmt.Println(len(c))

fmt.Println(cap(c))
```

Then we get as output:
```go
a=  [1 2 3]
b=  [1]
c=  [1 2]
1
3
2
3
```
*why*?
The underlying array used by b had a capacity of 3, if it had a capacity of 1, then 
we could not do what we did, and the program would die.
`d := c[0:1:1]`, using this notation we change the capacity. 

When I append to c, If it has 2 elements and it referenced a, I will change the 3'rd element of a.

*The conclusion*:
There are two things going on, but most importantly slices are aliases to arrays.
If we change any reference, all references values will change.
Doing an append will make a copy and give us the data into the new location.


