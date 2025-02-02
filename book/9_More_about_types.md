# Chapter 9 – More about types

## 9.1 First class Types
Types are first-class values in Jai. This means that you can do the same things with a type than you can do with any other value. This gives you a first glimpse of the flexibility of the language.  
All types in Jai have type **Type**, the type of Type is also Type.
**Code** is also a type (see § 26.4.5).

Besides the basic types mentioned in § 5 and § 6, Jai also has  other fundamental types, such as: string, pointer, proc (procedure), struct, array, and enum.

If a Type is constant (in other words, known at compile time),
you can declare other variables of that Type.  

The compiler has complete knowledge of all the types at compile-time, and some of that remains accessible at run-time. We'll later see (§ 26) how to use that.  
Here are some manipulations with types, explained in the next subsections:

See *9.1_types.jai*:

```c++
#import "Basic";

Thread_Index :: s64;  // (0A)

main :: () {
    // Type alias:
    t_index : Thread_Index = 100;
    TI :: s32; // (8B) TI is now a constant, of type Type, of value s32.
    s: TI = 7;
    t: TI = 9;
    print("s * t = %\n", s * t); // => 63
    print("size_of(TI) is %\n", size_of(TI)); // => 4 (bytes)    
    
    // type Type
    a: Type = float64;  // (1)
    b := int;           // (2)
    print("b is %\n", b); // => s64  (int is an alias for s64)
    c := type_of(42);   // (3)
    print("c is of type %, value %\n", type_of(c), c); // => c is of type Type, value s64
    d := type_of(b);    // (4)
    print("d is of type %, value %\n", type_of(d), d); // => d is of type Type, value Type
    var := Type;
    print("Type of Type is: %\n", type_of(var)); // => Type of Type is: Type
 
    // (5) size_of:
    print("the size of c is %\n", size_of(type_of(c))); // => 8
    t1 := 3.14; 
    print("the type of t1 is %\n", type_of(t1)); // => float32
    print("the size of t1 is %\n", size_of(type_of(t1))); // => 4
    print("the size of u16 is: % bytes\n", size_of(u16)); // => The size of u16 is: 2 bytes
    t2 := "Hello!";
    print("the type of t2 is %\n", type_of(t2)); // => string
    print("the size of t2 is %\n", size_of(type_of(t2))); // (6) => 16
    t3 := "世界!"; // (7) Chinese Hello World!
    print("the size of t3 is %\n", size_of(type_of(t3))); // => 16
    print("the size of Type is % bytes\n", size_of(Type)); // => 8 bytes
    
    // Any:
    x: Any = 3.0;  // (9)
    x = 3;         // (10)
    x = "Hello";
    x = 2.345;
    x = main;      // (11)
    print("the type of x is %\n", type_of(x)); // => the type of x is Any
    print("The type of main is %\n", type_of(main));  // (11B)
    // => The type of main is procedure ()

    a5 : s32 = 5;
    x2: Any = a5;
    print("the size of Any is %\n", size_of(Any));       // => 16

    // (12) Type comparisons:
    y : float64 = 0.1;
    assert(type_of(y) == float64);
    n := 42;
    assert(type_of(n) == int);
    assert(type_of(n) != string);
    print("is b a float? %\n", b == float); // => false
}
```

## 9.2 Constants of type Type: Type alias
An existing type can be given a new name (a **type alias**), defined as a constant, see Thread_index and TI in lines (0A) and (0B). Then you can define variables of the new type, and use these in other operations defined for the basic type. 
See also type variants: § 26.13.

## 9.3 Variables of type Type
In lines (1) to (4) we have four variables a to d that all have type Type; d even has value Type!  

## 9.4 size_of
**size_of** is a procedure that gives you the size in bytes a variable occupies in memory.
To use it, first apply type_of() to the variable:     **size_of(type_of(variable))**
See the examples starting in line (5):
t2 is of type string, and in line (6) we see that the size of a string is 16 bytes, no matter how many characters it contains; this is also the case for Unicode strings (see line (7)). This is because string is defined as a struct (see § 19.1).  
We see that Type itself is 8 bytes in size.

## 9.5 The Any type
(see line (9) and following)  
The **Any** type (defined in module _Preload_, see § 16.1), is the most wide type. It encompasses and matches with all other types. Values of all types can be converted to Any.
This allows you to use one variable to assign values of different types (see lines 10-11, even a procedure type as in line (11)), as you might do in a dynamically-typed programming language. Line (11B) tells us that the type of main is `procedure ()`.

The Any type has size 16 bytes (if you want to know why, see § 16.1). It is in fact a more informative and type-safe version of the void pointer from C, including a type pointer for the type and a void pointer for the value.
It is useful when working with heterogeneous arrays of pointers to different types.

## 9.6 Any and the print procedure
`print` is defined as:

```c++
print :: (format_string: string, args: .. Any, to_standard_error := false) -> bytes_printed: s64 {
//..
}
```

The print function takes in a format_string (which can also be a variable), and a variable (..) number of type Any parameters at compile time. That's why it can print out variables of any type. That type information is also available at runtime. 

## 9.7 Type comparisons
Types can be compared for equality with == and != : see line (12) and following.

