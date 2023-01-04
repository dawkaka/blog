Welcome,

In this article we're going to discuss all there is to know about arrays and slices in Go, we'll go deep talking about their internal structures and why they behave very differently even though they do similar things. We'll be going through Arrays and Slices and their behaviors and difference under 5 topics;

- Default or "zero" values
- Declaration and Initializing Arrays and Slices
- Value parts of Arrays and Slices
- Manipulating Arrays and Slices
- Potential quirks of slices
- Tips for optimizing performance in your code

**Things to note**

- `len` is a built-in function that returns the number of items in an array of slice
- `cap` is a built-in function that returns the capacity of an array or slice. The capacity of an array is equal to it's length and the capacity of a slice is the maximum number of elements that the slice can hold before it needs to be resized (slice capacity may be greater than the length)
- `fmt.Println` is a function in the `fmt` package in Go that is used to print a line to the standard output. It takes one or more values as arguments and prints them to the console, separated by spaces and followed by a newline.

##Default or "zero" values
In Go, if you declare a variable without explicitly initializing it, the variable is automatically set to the zero value for its type. The zero value is a default value assigned to a variable of a specific type when it is declared but not explicitly initialized. For example, if you declare an int variable like this:

```go
var x int
```

The value of x will be initialized to 0.
In other languages like Javascript value of an uninitialized variable is `undefined`

The zero value for each type is as follows:

- `int`: 0
- `float`: 0.0
- `bool`: false
- `string`: "" (empty string)
- `pointer`: nil
- `struct`: all fields are set to the zero value for their respective types

The zero value of an array in Go is an array with all elements set to the zero value for their respective types. For example, if you have an array of integers:

```go
var arr [5]int
```

The zero value of this array would be:

```go
[0, 0, 0, 0, 0]
```

Similarly, if you have an array of strings:

```go
var arr [5]string
```

The zero value of this array would be:

```go
["", "", "", "", ""]
```

In Go, the zero value of a slice is nil, which is a slice with a length of 0, a capacity of 0, and no underlying array. For example:

```go
var slice []int
fmt.Println(slice == nil) // => true
```

##Declaring and Initializing Arrays
The format for declaring arrays in Go is `var name [L]T`.
`var` is a keyword for declaring all kinds of variables in Go
`name` is the name of the variable, can be anything
`L` is the length of the array (must be a constant) and `T` is the type of the array items.
Let's look at some examples

```go
//Array of 5 Intergers
var nums [5]int
fmt.Println(nums) // => [0 0 0 0 0]

//Array of 10 strings
var strs [10]string
fmt.Println(nums) // => [         ]

// Nested arrays
var nested = [3][5]int{
	{1, 2, 3, 4, 5},
	{6, 7, 8, 9, 10},
	{11, 12, 13, 13, 15},
}
fmt.Println(nested) // => [[1 2 3 4 5] [6 7 8 9 10] [11 12 13 13 15]]
```

Initializing arrays is simply assigning a value to the variable
`var name = [L]T{...}` where `...` represents the array items of type `T`

```go
//Intializing an array containing 10 intergers
var nums = [10]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
fmt.Println(nums) // => [1 2 3 4 5 6 7 8 9 10]

//Intializing an array containing 10 strings
var strs = [10]string{"one", "two", "three", "four", "five", "six", "seven", "eight", "nine", "ten"}
fmt.Println(strs) // => [one two three four five six seven eight nine ten]

//Nested arrays
var nested = [3][2]int{}
```

You can also create an array of structs

```go
type Car struct {
	Brand string
	Color string
	Price float32
}

//Array of 5 items of type Car
var arrayOfCars = [5]Car{
	{Brand: "Porsche", Color: "Black", Price: 20_000.00},
	{Brand: "Volvo", Color: "White", Price: 8_000.00},
	{Brand: "Honda", Color: "Blue", Price: 7_000.00},
	{Brand: "Tesla", Color: "Black", Price: 50_000.00},
	{Brand: "Kia", Color: "Red", Price: 5_000.98},
}
fmt.Println(arrayOfCars) // => [{Porsche Black 20000} {Volvo White 8000} {Honda Blue 7000} {Tesla Black 50000} {Kia Red 5000.98}]
```

To create an array of items with different types in Go, you can use the `interface{}` type.
An interface in Go is a type that defines a set of methods that a type must implement. Any type that implements all of the methods listed in an interface is said to satisfy the interface and is considered to be of that interface type. The special interface type `interface{}` has no methods, which means that every type in Go satisfies this interface.

```go
package main

import "fmt"

func main() {
	//Array containing 5 items of different type
	var randomsArray = [5]interface{}{"Hello world!", 35, false, 33.33, 'A'}
	fmt.Println(randomsArray) // => [Hello world! 35 false 33.33 65]

}
```

Other ways of initializing arrays

```go
import "fmt"

func main() {
	// Using shorthand syntax
	cars := [3]string{"Tesla", "Ferrari", "Benz"}
	fmt.Println(cars) // => [Tesla Ferrari Benz]

	// Using ... inplace of array length
	digits := [...]int{10, 20, 30, 40}
	fmt.Println(digits) // => [10 20 30 40]

	// Using len keyword
	countries := [len(digits)]string{"China", "India", "Kenya"}
	fmt.Println(countries) // => [Chian India Kenya]
}
```

**Note** you can't use the `:=` shorthand syntax in global scope

##Declaring and Initializing Slices
To declare a slice we use the format `var name []int`, the only difference between declaring arrays and slices is that with slices we omit the length.
Examples

```go
import "fmt"

func main() {
	// A slice of intergers
	var intSlice []int
	fmt.Println(intSlice) // => []

	// A slice of intergers
	var stringSlice []string
	fmt.Println(stringSlice) // => []
}
```

To initialize a slice in Go, you can use the make function. The make function takes three arguments: the type of the slice, the length of the slice, and the capacity of the slice (which is optional). `make([]T, len, cap)`

For example, to create a slice of integers with a length of 5 and a capacity of 10, you can use the following code:

```go
package main

import "fmt"

func main() {
	// With capacity
	slice1 := make([]int, 5, 10)
	fmt.Println(len(slice1), cap(slice1)) // => 5 10

	// Without capacity
	slice2 := make([]int, 5)
	fmt.Println(len(slice2), cap(slice2)) // => 5 5
}
```

When you omit the capacity, the capacity is set to the length of the slice.

You also initialize a slice without the `make` function by immediately assigning value to it

```go
slice := []int{1, 2, 3}
fmt.Println(len(slice), cap(slice)) // => 3 3
```

##Value parts of Arrays and Slices
This is the most important difference between arrays and slices in Go, Arrays have only one part whiles slices have direct and indirect parts (2 parts). What this means is that arrays in Go are fixed-length data structures that consist of a contiguous block of memory for storing elements. Slices are dynamically-sized and reference a contiguous segment of an underlying array.
To understand this properly, let's look at the value parts of the this array and slice example

```go
var arr = [5]int{1,2,3,4,5}
var slice = []int{1,2,3,4,5}
```

array
![in memory representation of array in Go](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/491zvin1v4flhyhc8md2.png)
'
'
slice
![in memory representation of array in Go](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/wa0m10rq8cvy7yejnznn.png)

From the above diagrams we can see that an array is a fixed-size collection of elements of the same type, stored in contiguous memory locations. On the other hand, a slice consists of a pointer to an underlying array, a length, and a capacity.

internal structure of a slice direct part

```go
type _slice struct {
	// referencing underlying elements
	elements unsafe.Pointer
	// number of elements
	len int
	// capacity of the slice
        cap int
}
```

##What happens when you copy Arrays and Slices
In Go, underlying value parts are not copied in value assignments only direct values a copied. What this means is that when we copy an array we're making a copy of it's elements (because it's has only a direct part) but when we copy a slice, we are making a copy of it's direct part i.e `len`, `cap` and `pointer to elements` and the indirect part (the actual elements are not copied)

####Array copy example

```go
x := [5]int{3, 6, 9, 12, 15}
y := v
```

In the above code example we've initialized an array `x` and then we created another variable `y` and we assigned (copied) the value of `x` to `y`.

Here is a diagram representation of what happens

![array copies creates new array](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tk21oocbqtl51od7wfff.png)
when we make a copy of an array all element are copied into a separate memory block. In the above code example, if we make changes to to `x` it doesn't affect `y` and the vice versa, we'll talk more on that later

####Slice copy example

```go
x := []int{2,4,6,8,10}
y := x
```

In the above code example we've initialized a slice `x` and then we created another slice `y` and we assigned (copied) the value of `x` to `y`.

![in memory representation of a slice](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bhyjsz6pa1l4dpv70eki.png)

From the above diagram you can see that the direct parts of `x` is copied into a separate memory for `y` (Len, Cap, and Elem pointer) but `x` and `y` still share the same underlying parts (array elements). So when you modify an element of `x` it also affects `y` since the share the same underlying elements, but `x` and `y` can have different lengths and capacity since those are in their own separate memories
more on this later

##Manipulating Arrays and Slices
In this section, we will discuss manipulating arrays and slices.
####Arrays
Since arrays in Go have a fixed constant length, the only manipulation that can be done on arrays is changing values at specific indexes starting from zero.
first element is `0`
second element is `1`
third element is `2` and so on.
Examples

```go
package main

import "fmt"

func main() {
	var fruits [6]string // Declare string array with zero values
	fmt.Println(fruits)  // => [       ] (string zero value "")

	// 🍊 change first element
	fruits[0] = "Orange"
	fmt.Println(fruits) // => [Orange      ]

	//🍋 change last element
	fruits[5] = "Lemon"
	fmt.Println(fruits) // => [Orange     Lemon]

	// 🍌🍉🍐🍏change all
	fruits[1] = "Banana"
	fruits[2] = "Watermelon"
	fruits[3] = "Pear"
	fruits[4] = "Apple"
	fmt.Println(fruits) // => [Orange Banana Watermelon Pear Apple Lemon]

	//🍍 Dont' like oranges? change first element again
	fruits[0] = "Pineapple"
	fmt.Println(fruits) // => [Pineapple Banana Watermelon Pear Apple Lemon]

	//Modify array of integers
	evenNumbers := [5]int{2, 4, 6, 8, 10}

	evenNumbers[0] = 12
	fmt.Println(evenNumbers) // => [12 4 6 8 10]

	evenNumbers[3] = 20
	fmt.Println(evenNumbers) // => [12 4 6 20 10]

}
```

Accessing array values

```go
import "fmt"

func main() {
	nums := [7]int{1, 2, 3, 4, 5, 6, 7}

	// get the first element
	first := nums[0]
	fmt.Println(first) // => 1

	// get third element
	fmt.Println(nums[2]) // => 3

	// last
	fmt.Println(nums[6]) // => 7

	//alternatively
	fmt.Println(nums[len(nums)-1]) // => 7

}
```

If we try to change element an index greater than or equal to the arrays length the code will won't compile it'll `panic` with an index out of bound error

```go
package main

import "fmt"

func main() {
	nums := [7]int{1, 2, 3, 4, 5, 6, 7}
	outOfBound := nums[7]
}
```

```
invalid argument: array index 7 out of bounds [0:6]
```

####Slices
Slices are a useful data type in Go because they offer a flexible and convenient way to manipulate collections of data. They can be accessed and modified in the same way as arrays, but they also have some specific behaviors that make them more powerful. In the following sections, we will explore some of these behaviors in more detail.
#####Slice expression
The slice expression signature is `s[start:end:cap]`
which is used to create a new slice that includes all the elements of the original slice `s`, starting from the element at index `start` and up to but not including the element at index `end`, the `cap` is the capacity of the newly created sublice and it's optional. If `cap` is omitted, the capacity of the sub slice is equal to it's length. The length of the sub slice is `end` - `start`

Example

```go
package main

import "fmt"

func main() {
	slice := []int{1, 2, 3, 4, 5, 6}
	subSlice := slice[1:4]
	fmt.Println(subSlice)                     // => [2 3 4]
	fmt.Println(len(subSlice), cap(subSlice)) // => 3 3

	subSliceWithCap := slice[1:4:5]
	fmt.Println(subSliceWithCap)                            // => [2 3 4]
	fmt.Println(len(subSliceWithCap), cap(subSliceWithCap)) // => 3 4
}
```

**note** the capacity also starts from zero, so if you use `5` as capacity of a slice `s` calling `cap(s)` will return `4`

If the `start` index is zero, you can omit it `s[:end]` similarly if the `end` index is the end of the array you can omit it like so `s[start:]`
Examples

```go
package main

import "fmt"

func main() {
	s := []string{"g", "o", " ", "i", "s", " ", "s", "w", "e", "e", "t"}

	// copy from 0 to index 2 (index 2 is exclusive)
	goSubSlice := s[:2]
	fmt.Println(goSubSlice) // => [g o]

	// copy from index 3 to end
	isSweetSubSlice := s[3:]
	fmt.Println(isSweetSubSlice) // => [i s   s w e e t]

        // copy all items
        copySlice := s[:]
        fmt.Println(copySlice) // => [g o  i s   s w e e t]
}
```

Earlier we discussed slice value parts and how only the direct parts of a slice is copied, so when we create a sub slice what is actually happening ?
Let's use this examples

```go
package main

import "fmt"

func main() {
	n := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
	n1 := n[:6]
	n2 := n[3:8]
	n3 := n[4:10]

	fmt.Println(n1, len(n1), cap(n1)) // => [1 2 3 4 5 6]  6 10
	fmt.Println(n2, len(n2), cap(n2)) // => [4 5 6 7 8]  5 7
	fmt.Println(n3, len(n3), cap(n3)) // => [5 6 7 8 9 10] 6 6

	// change n1 at index 4 to 15
	n1[4] = 15

	fmt.Println(n)  // => [1 2 3 4 15 6 7 8 9 10]
	fmt.Println(n1) // => [1 2 3 4 15 6]
	fmt.Println(n2) // => [4 15 6 7 8]
	fmt.Println(n3) // => [15 6 7 8 9 10]

}
```

Notice how when we changed the `n1[4]` to 15 it affected all the other sub slices including the main slice? that's because they share the same underlying array elements, so anytime we make changes to a sub slice it's affects all other sub slices.
Here is a diagram of the above slice and sub slices to help you understand what's happening

![Slices sharing the same underlying arrays but cover different parts](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vqxpfrrbuemjrcxlu0vo.png)

From the above diagram we can see all sub slices share the same underlying array but they span different parts of the underlying array. When we make a change at a particular index of a sub slice, all sub slices the span that index are are also modified.
When you modify the element at index 4 of `n3`, you are also modifying the element at index 8 of the array (`n`), because `n3` and n share the same underlying array. As a result, the change to `n3[4]` is also reflected in `n`. Similarly, any changes made to the elements of `n` between indices 4 and 9 (inclusive) will also be reflected in `n3`, because `n3` spans those indices.

```go
n3[4] = 18
fmt.Println(n)  // => [1 2 3 4 15 6 7 8 18 10]
fmt.Println(n1) // => [1 2 3 4 15 6]
fmt.Println(n2) // => [4 15 6 7 8]
fmt.Println(n3) // => [15 6 7 8 18 10]
```

######Appending items to a slice
The Go append function allows you to add elements to the end of a slice. It has the following syntax:

```go
func append(s []T, x ...T) []T
```

s is the slice you want to append to, `x` is a list of one or more elements of type `T` to be appended, and the function returns a new slice with the appended elements.

For example:

```go
s := []int{1, 2, 3}
s = append(s, 4, 5, 6)
fmt.Println(s) // => s is now [1, 2, 3, 4, 5, 6]
```

Note that if the capacity of the underlying array is insufficient to accommodate the additional elements, append will allocate a new, larger array to hold the result. If a new larger array is created after append, that slice does no longer share the same underlying array with other sub slices
Let's use the previous example

```go
package main

import "fmt"

func main() {
	n := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
	n1 := n[:6]
	n2 := n[3:8]
	n3 := n[4:10]

	fmt.Println(n1, len(n1), cap(n1)) // => [1 2 3 4 5 6]  6 10
	fmt.Println(n2, len(n2), cap(n2)) // => [4 5 6 7 8]  5 7
	fmt.Println(n3, len(n3), cap(n3)) // => [5 6 7 8 9 10] 6 6

}
```

from the above code `n2` has a length of 5 and a capacity of 7 which means we can append two more items without a new array being created and it'll continue so share the same underlying array with the other sub slices
See

```go
n2 = append(n2, 100)
fmt.Println(n)  // => [1 2 3 4 15 6 7 8 100 10]
fmt.Println(n1) // => [1 2 3 4 15 6]
fmt.Println(n2) // => [4 15 6 7 8, 100]
fmt.Println(n3) // => [15 6 7 8 100 10]

n2 = append(n2, 101)
fmt.Println(n)  // => [1 2 3 4 15 6 7 8 100 101]
fmt.Println(n1) // => [1 2 3 4 15 6]
fmt.Println(n2) // => [4 15 6 7 8 100 101]
fmt.Println(n3) // => [15 6 7 8 100 101]

// Check the capacity and length of n2
fmt.Println(cap(n2), len(n2)) // =>  7 7
```

As we appended more items it was affecting `n` and `n3` but now the `n2` slice is full (capacity == length).
Now that capacity of n2 is equal to it's length so appending a new item will cause new array to created for n2 and it will no longer share the same underlying array with the other sub slices

```go
n2 = append(n2, 102)
fmt.Println(n)  // => [1 2 3 4 15 6 7 8 100 101]
fmt.Println(n1) // => [1 2 3 4 15 6]
fmt.Println(n2) // => [4 15 6 7 8 100 101 102]
fmt.Println(n3) // => [15 6 7 8 100 101]
```

From the above code you can see only `n2` was modified and the others weren't affected.
######appending multiple items to a slice
example

```go
package main

import "fmt"

func main() {
	s := []int{10, 20, 30, 40, 50, 60}
	s2 := []int{70, 80, 90}

        // Appending slice to slice
	s = append(s, s2...)
	fmt.Println(s) // => [10 20 30 40 50 60 70 80 90]

        // Appending multiple values
	s = append(s, 100, 110, 120)
	fmt.Println(s) // => [10 20 30 40 50 60 70 80 90 100 110 120]
}
```

######Deep value copying of a slice
Deep copying simply means copying the underlying array of a slice instead of the direct part so the destination slice doesn't share the same underlying memory with the source slice.
deep copying using append

```go
package main

import (
	"fmt"
)

func main() {
	slice1 := []int{1, 2, 3, 4, 5, 6}
	slice2 := []int{}
	slice2 = append(slice2, slice1...)

	fmt.Println(slice1) // => [1 2 3 4 5 6]
	fmt.Println(slice2) // => [1 2 3 4 5 6]

	//Modifying slice2 doesn't affect slice1
	slice2[0] = 100
	fmt.Println(slice1) // => [1 2 3 4 5 6]
	fmt.Println(slice2) // => [100 2 3 4 5 6]

	// copying a range of items
	slice3 := []int{}
	slice3 = append(slice3, slice1[3:5]...)
	fmt.Println(slice3) // => [4 5]

	//Again slice3 and slice1 doesn't share underlying array

	slice3[0] = -10
	fmt.Println(slice1) // => [1 2 3 4 5 6]
	fmt.Println(slice3) // => [-10 5]

}
```

In Go, you can use the copy function to perform a deep copy of a slice. A deep copy creates a new slice with its own, independent copy of the elements of the original slice.

The copy function has the following syntax:

```go
func copy(dst, src []T) int
```

`dst` is the destination slice, and `src` is the source slice. Both slices must have the same element type `T`. The function returns the number of elements copied, which will be the minimum of the lengths of `dst` and `src`.

For example:

```go
package main

import "fmt"

func main() {
	s := []int{1, 2, 3}
	t := make([]int, len(s))
	copy(t, s)
	fmt.Println(t) // => [1, 2, 3], and is a deep copy of s

	// copy a range of items

	t = make([]int, len(s)-1)
	copy(t, s[0:2])
	fmt.Println(t) // => [1, 2], and is a deep copy of s

}
```

Note that if the length of `dst` is less than the length of `src`, only the first `len(dst)` elements of `src` will be copied. To make a deep copy of the entire slice, you must ensure that `dst` has sufficient capacity to hold all of the elements of `src`.

##Potential quirks of slices
As mentioned earlier, a sub slice only copies the direct part of a slice and share the same underlying array. So when we have a slice `sBig` of size 10 mega bytes and we create a sub slice `sTiny` of size 3 bytes from `sBig`, `sTiny` and `sBig` will reference the same underlying array. As you might know Go is garbage collected which means it automatically frees up memory when it is not referenced (it can not be reached).
So in the case of `sBig` and `sTiny` even if we only need `sTiny` which is 3 bytes, `sBig` will continue to be in memory because `sTiny` references the same underlying array as `sBig`. To get around this we make a deep copy so that `sTiny` doesn't share the underlying array as `sBig` and so it can be garbage collected thereby freeing up memory.
Example

```go
var gopherRegexp = regexp.MustCompile("gopher")

func FindGopher(filename string) []byte {
    //Reading a very huge file  1,000,000,000 bytes (1GB)
    b, _ := ioutil.ReadFile(filename)
    //Taking a just 6 byte sub slice
    gopherSlice := gopherRegexp.Find(b)
    return gopherSlice
}
```

From the above example we read a very huge file (1GB) and returned a sub slice of it (just 6 bytes), since the `gopherSlice` still reference the same underlying array as the huge file, which means that 1GB of memory can not be garbage collected even though we are not using it anymore.
If you call the `FindGopher` function multiples times, you program can eat all the computers memory. To fix this, like I said earlier we make a deep copy so `gopherSlice` doesn't share the same underlying array as the huge slice
Example

```go
var gopherRegexp = regexp.MustCompile("gopher")

func FindGopher(filename string) []byte {
    //Reading a very huge file  1,000,000,000 bytes (1GB)
    b, _ := ioutil.ReadFile(filename)
    //Taking a just 6 byte sub slice
    gopherSlice := make([]byte, len("gopher"))

    // Make a deep copy
    copy(gopherSlice, gopherRegexp.Find(b...)
    return gopherSlice
}
```

Now the Go language garbage collector can now free up the ~1GB of memory

##Tips for optimizing performance in your code
Like I said earlier the most important difference between arrays and slices in Go is the difference in their value parts this together with Go value copy cost is the reason they differ in performance.

#####Value copy cost
Value assignments, argument passing, looping using `range` keyword etc. all involve value copying. The bigger the value size the bigger the value copy cost, copying 10 mega bytes will take longer than copying 10 bytes. And we also learned that only direct parts are copied.
Example

```go
array := [100]int{1,2,3,4,5,6, ..., 100}
slice := []int{1,2,3,4,5,6, ..., 100}
```

From the above example we've created an array of numbers `1-100` and a slice also of numbers `1-100`. When we copy the array all the elements are copied so the value copy cost will be `8  * 100 = 800 bytes` (1 `int` is `8 bytes` assuming 64-bit architectures) but when we copy a slice only the direct part is copied (`len`, `cap` and `elements pointer`) so the value copy cost will be `8 + 8 + 8 = 24 bytes`. Even though bough slice and array contain 100 elements the value copy cost of the array is way bigger than that of the slice.

From the above scenario the performance concerns only affects arrays and not slices, I'll be focusing on how to use arrays with performance in mind. Also for small array sizes these performances differences are negligible and not worth the effort to make things faster

The only thing here is not using `range` key word to loop over arrays

```go
package main

import "fmt"

func main() {
	// Don't do this
	arr := [10]int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}

	// arr is copied
	for key, value := range arr {
		fmt.Println(key, value)
	}

	// Do this instead
	for i := 0; i < len(arr); i++ {
		fmt.Println(i, arr[i])
	}
}
```
