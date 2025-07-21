****### Class 12: Structs, Tags and JSON

 *Structural Compatibility*:
 Even though the structs are anonymous, and have no type defintions,
 if they contain the same value declarations and methods, then one can be assigned to another. (copied)

But if the structs are named differently, even though they may have the same types and functions inside, then they cannot be assigned to one another, but again, they can be converted from one to another.

**The Rules**:
Two structs are compatible if:
- the fields have the same types and names
- in the same order
- and the same  (even though the tag name may be different, struct conversion can be performed)

- A struct is comparable if all the fields inside are comparable
- A struct may be copied or passed as a parameter in it's entirety
- The zero value for a struct is "zero" for each field in turn
- Structs are passed by values, unless a pointer is used, not like slices

```go

var white album

func SoldAnother(a album){
	a.copies++
	// OOPS
}

func SoldAnother(a *album){
	a.copies++
	// what we wanted
	// this is equals to (*a).copies++
	// go does this inherently
}

```

When initialising structs, the law states that it is usually desirable that the zero value be a natural or sensible default.

Means, even if no value is assigned to the struct it can be used, because the default values are set up in such a way that there would be no error

Ex.

```go

type Buffer struct {
	buf []byte      // contents are the bytes buf[off :len(buf)]
	off int         // read at &buf[off] write at &buf[len(buf)]
	lastRead readOp // last read operation, so that Unread* can work correctly 
}

```

- A struct with no fields is useful, as it takes up no space

```go
var isPresent map[int]struct{}
done := make(chan struct{})
```

The `struct{}` being referred to in both the cases is the same, go actually keeps the empty struct as a singleton in each program.

*Why does json tags exist*?

```go

var Response struct{
	Page int        `json:"page"`
	words []string  `json:"words`"
}
```

This struct will not compile because, the *words* is lowercase, and thus go will not export it. Private fields of a struct that are not exported are not encoded.
The complier won't complain but the linter will give a warning that this is not what we intended.


### Reference and Value semantics

When to use pointers, when to use values, and what are the tradeoffs and side effects?

- The intention of using pointers is to share
- Not sharing is better with concurrency

*Why to use pointers*?
- Some objects can't be copied safely (mutex)
- Some objects are too large to copy efficiently  ( > 64 bytes)
- Some methods need to change (mutate) the receiver
- When decoding protocol data into an object
- When using a pointer to signal a 'null' object
- Wait groups(in concurrency) have to be passed as references

**Mutex cannot be copied, hence we have to use pointers**
 Because the intention of using mutex, is that we wished to share something

NOTE: If you passed something via a pointer, make sure that you always pass it by a pointer (I mean if you have multiple functions for the same DS) because the changes may be then lost if you pass it via the value

*Stack allocation*:
- Is more efficient
- Accessing a variable directly is more efficient than following a pointer
- Accessing a dense sequence of data is more efficient than sparse (Ex. Array as compare to a LL)

*Sometimes you just can't allocate on the stack*?
- A function returns a pointer to a local object
- A local object is captured in a function closure
- A pointer to a local object is sent via a channel
- any object is assigned into an interface
- any object whose size is variable at runtime (slices)

NOTE: things created by new might sometimes end up in stack and sometimes on the heap so it's better to avoid it altogether

```go

for i,value : range things{
	// value is a copy and thus any change made to it won't be visible outside the for loop
}

for i := range things{
	things[i].someKey = changeit
	// now this change is visible
}

func x(thing []someDS) someDS{
	// modification to ds
}

sliceVar = append(sliceVar,[]{someVals})
```

In all these cases, we can have the original value relocation, hence to the variable we need to again assign the new value

```go
alice := &users[0] // RISKY

// this users is a slice, and thus it's value's address are not permanent. Assigning that address to another var to access it is a risky operation because it might lead to errors
```


### Networking with HTTP

The handler is handling multiple parallel requests based on how many our hardware can support, so no need to perform parallelism by ourself

*The design of GO's HTTP*
An HTTP handler function is an instance of an interface

```go
type Handler interface{
	serveHTTP(ResponseWriter, *Request)
}

type HandlerFunc func(ResponseWriter, *Request)

func (f HandlerFunc) ServerHTTP(w ResponseWriter,r *Request){
	f(w,r)
}
```

Go doesn't have classes, but allows methods to be put into any user declared types, so I can create a method for a function, and the method runs on the function just by calling the function

- When doing `json.Unmarshal(response,&SomeStruct)` you need not define all the fields of the response in the struct, and you can only define those values that you want

*Avoid json.Unmarshal*:

```go
err = json.NewDecoder(resp.Body).Decode(&item); err!=nil{
	http.Error(w,err.Error(),http.StatusInternalServerError)
	return
}
```
We can do this instead

### OOPs in Go

OOP is always boiled down into 4 parts:
- Abstraction
- Encapsulation
- Polymorphism
- Inheritance

Sometimes the last two are very confusing

*Abstraction*:
Decoupling the behaviour from the implementation details
Ex. The UNIX file system API is a good example
Roughly five basic functions hide all the messy details:
- Open
- Close
- Read
- Write
- ioctl

*Encapsulation*:
Hiding the implementation details from misuse
Encapsulation usually means controlling the visibility of names

It's hard to maintain abstraction if the details are exposed
- The internals may be manipulated in ways contrary to the concept behind the abstraction
- Users of the abstraction may come to depend on the internal details - but those might change'

*Polymorphism*:
Means many shapes - multiple types behind a single interface

Three main types are recognised:
- ad-hoc: typically found in function/operator overloading
- parametric: commonly known as "generic programming"
- subtype: subclasses for substituting for superclasses

Protocol oriented programming uses explicit interface types (ad-hoc)
 
Behaviour is complete separate from implementation, which is good for abstraction

*Inheritance*:
It has conflicting meanings:
- Substitution (subtype) polymorphism
- Structural sharing of implementation details

In theory, inheritance should always imply subtyping
the subclass should be a "kind of" superclass

Inheritance can be bad?
- What if the superclass changes behaviour
- What if the abstract concept is leaky

Not having inheritance means better encapsulation and isolation
Have *composition* instead

*How does Go do it*?
- Encapsulation using packages for visibility control
- Abstraction and polymorphism using interface types
- Enhanced composition to provide structure sharing

Go does not offer inheritance or substitutability based on types
**Substitutability is based only on interfaces: purely a function of abstract behaviour**

You can define methods on any user-defined type, rather than only a 'Class'
Go allows any object to implement the methods() of an interface, not just a subclass

### Methods and Interfaces

*An interface specifies abstract behaviour in terms of methods*
```go
type Stringer interface {
	String() string
}
```
Concrete types offer methods that satisfy the inheritance

*A method is a special type of function*
It has a *receiver* parameter before the function name parameter

```go
type IntSlice []int 

func (is IntSlice) String() string {

var strs []string

for _,v := range is {
strs = append(strs,strconv.Itoa(v))
}

return "[" + strings.Join(strs, ";") + "]"
}
```
In this case the receiver is the `is` which is an instance of the IntSlice type s

*Why interfaces*?
Without interfaces we would have to write (many) functions, 
for (many) concrete types, possibly coupled to them

Better- We want to define our functions in terms of abstract behaviour

```go
type Writer interface{
	Write([]byte) (int,error)
}

func OutputTo(w io.Writer,...) {....}
```

So what does OutputTo take?
It takes any Object that provides a write method

An interface specifies the required behaviour as a method set
, and then we have an actual type that implements that method set

*Why does it look so confusing*
*In other languages when you implement an interface you have to say that.*
Because we never read the work `implements` anywhere, GO doesn't do that
The way we think about interfaces in go is from the consumer's side and not the provider's side
I, the consumer, want a particular interface that provides a particular method that provides this particular behaviour

Any type that implements that method set satisfies the interface

```go

type Stringer interface {
	String() string
}

func (is IntSlice) String() string {

}
```
This is k.a. structural typing
No type will declare itself to implement `ReadWriter` explicitly

Methods may be declared on any `user-declared type`
Ex.
```go
type Myint int
func (i MyInt) String() string{

}
```
The same method name may be bound to different types

A method may take a pointer or value receiver, but not both
```go

type Point struct {
	X, Y, float64 
}

func (p Point) Offset(x, y, float64) Point{
	return Point{p.x+x,p.y+y}
}

func (p *Point) Move(x, y float){
	p.x+=x
	p.y+=y
}
```
Taking a pointer allows the method to change the receiver (original object)

NOTE: All the methods must be present to satisfy the interface

```go

var w io.Writer
var rwc io.ReadWriteCloser

w = os.Stdout             // OK: *os.File has Write method
w = new(bytes.Buffer)     // OK: *bytes.Buffer has Write method
w = time.Second           //ERROR: no write method

rwc = os.Stdout // OK: *os.File has all three method
rwc = new(bytes.Buffer) // ERROR: no Close method

w = rwc // OK: io.ReadWriteCloser has Write
rwc = w // ERROR : no Close method 
```

Because of the last 2 lines, it is better to keep the writer small

NOTE:
The Receiver must be of the right type (pointer or value)
 ```go

type IntSet struct {}

func (*IntSet) String() string

var _ = IntSet{}.String() // WE cannot do this, as this is a literal
// Why? Because String needs *IntSet
// and this initialisation has no place in the memory

var s IntSet
var _ = s.String() // OK

var _ fmt.Stringer = &s  //OK
var _ fmt.Stringer = s   //ERROR: no String method
// Because we defined our stringmethod to take the pointer receiver, (it will only work on the pointer receiver)
// SO the other way won't work??
// COMPLICATED: COME BACK LATER
// we need to have the actual address of the object to make the stringer happy 
```

*Interface Composition*
We can compose our interfaces, without defining the methods in the big interface.
```go

type reader interface{
	Read(p []byte) (n int,err error)
}

type writer interface{
	Wtite(p []byte) (n int,err error)
}

// then we have:

type ReadWriter interface{
	Reader
	Writer
}
```

*What is the reason for this*?
Usually interfaces are supposed to be kept small.

And this is related to the extension of types. 
In Golang, all methods for a given type must be declared in the same package where the type is declared.
This allows a package importing the type to know all the methods at compile time.
**But**, we can always extend the type in a new package through embedding.
```go
type Bigger struct {
	my.Big     // get all Big methods via promotion
}
func (b Bigger) // New method added in the same file/package
```

```go
// check out supporting code in goCode
```

### Class 19:Struct Composition (This is not inheritance)

The fields of an embedded struct are promoted to the level of the embedding structure.

```go
type Pair struct {
	Path string
	Hash string
}

type PairWithLength struct {
	Pair  // the fields for Pair appear in the same level as Length 
	Length int
}
// It means that p1.Path makes sense
// Makes the same sense as p1.length
func (p Pair) String() string {
return fmt.Sprintf("Hash of %s is %s.",p.Path,p.Hash)
}
func testStructComposition() {
p := Pair{"abc","Ocedf"}
p1 := PairWithLength{Pair{"ana","kaubd"},1212}
fmt.Println(p)
fmt.Println(p1)
}

// this case has ouput of 
// Hash of abc is Ocedf
// Hash of ana is kaubd
```
Because the PairWithLength didn't have a string method, the string method of Pair was promoted to PairWithLength and we used it.
But if we define a `String() string` method for PairWithLength then that would be used instead.
*Do not confuse this with overriding*
Because we are not doing inheritance.
```go
func FileName(p Pair) string{
return filepath.Base(p.Path)

// Hence
fmt.Println(FileName(p1)) 
// Won't work
```
Then in the fileName function, we cannot pass PairWithLength even though it contains the Pair, because this is not inheritance

A struct can also embed a pointer to another type; promotion of it;s fields and methods work the same way
```go
type FigZig struct {
	*PairWithLength
	Broken bool
}

fg := FigZig{
	&PairWithLength{Pair{"/usr","Oxfdfe"},121},
	false,
}
// then 
fmt.Println(fg)
// Will use the string function of PairWithLength
```

*Let's look at a practical case to understand why this is helpful*
*We will see how `Sort.Interface` is implemented*

```go

type Interface interface {
	Len()// no of elements in collection
	Less(i, j int) bool//how to check
	Swap(i, j int)//how to swap
}

func sort.Sort(data Interface)
```

We have built-ins for sorting.
Ex. Slices of strings can be sorted using StringSlice.
```go
entries := []string{"Charlie","able","Dog"}
sort.Sort(sort.StringSlice(entries))
```
So, what this StringSlice function does it that it takes the string and casts it into a type which has those three methods defined.
And now because that type has that methods defined it implements that interface.

*Now look at another use case
`sort.Reverse`*

```go

type Reverse struct{
	Interface
}
// now all methods of inteface are promoted to reverse

// and what reverse does is that it overwrites one of the methods

func(r reverse) Less(i, int j) bool{
	return r.Interface.Less(j,i)
}

func Reverse(data Interface) Interface {
	return &reverse{data}
}

// How does it work?

sort.Sort(sort.Reverse(sort.StringSlice(entries)))

```

*Making the nil useful*

```go

type StringStack struct {
	data []string
	// 'd' in data is small so we don't expose the internal
	// data structure
}

func (s *StringStack) Push(x string){
	s.data = append(s.data,x)
}

func(s *StringStack) Pop() string {
	if l:= len(s.data); l>0 {
		t:= s.data[l-1]
		s.data := s.data[:l-1]
		return t
	}
	panic("pop from the empty stack")
}
```

Nothing in go Prevents calling a method with a `nil` receiver
In this case we called on the last element, handled the `nil` by ourselves.


### Class 20: Interfaces and Methods in Details
We will cover some intricate details and some pragmatic ways for how to use interfaces.