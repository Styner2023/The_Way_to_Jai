# Chapter 16 - Types in depth

At compile time Jai deduces all there is to know about the types of all variables and objects.  
This info is heavily used in meta-programming at compile-time, and it is even (partially ??) known at run-time.
In this chapter we'll see which data-structures from built-in modules are used for this processing. Most of these are defined in module _Preload_.

>To see how type inference works in depth, read the masterfully written 090_how_typechecking_works article in the how_to/ folder of the compiler.)

## 16.1 Definition of Any
We encountered the Any type in § 9.4, it is defined as follows:   

```c++
Any_Struct :: struct {  // This is what an Any looks like.
    type: *Type_Info;
    value_pointer: *void;
}
```

It is a struct of two values: the type info and the pointer to the value (a *void because we don't know the type of the value, that's why its Any!).  
What is this Type_Info ?

## 16.2 Type_Info and Type_Info_Tag
This is its definition:

```c++
Type_Info :: struct {
    type:           Type_Info_Tag; 
    runtime_size:   s64;
}
```

The `type` field is of type `Type_Info_Tag`, the 2nd field is the size in bytes.

`Type_Info_Tag` itself is an enum:

```c++
Type_Info_Tag :: enum u32 {
    INTEGER              :: 0;
    FLOAT                :: 1;
    BOOL                 :: 2;
    STRING               :: 3;
    POINTER              :: 4;
    PROCEDURE            :: 5;
    VOID                 :: 6;
    STRUCT               :: 7;
    ARRAY                :: 8;
    OVERLOAD_SET         :: 9;
    ANY                  :: 10;
    ENUM                 :: 11;
    POLYMORPHIC_VARIABLE :: 12;
    TYPE                 :: 13;
    CODE                 :: 14;
//    UNARY_DEREFERENCE :: 15;
//    UNARY_LITERAL :: 16;
    VARIANT              :: 18;
}
```
It literally enumerates all types than can occur in Jai.  
Then for nearly every type, there is a struct which comprises a Type_Info struct, as well as some other useful info for that specific type, like if it is signed or not for an integer. Here is an example:

```c++
Type_Info_Integer :: struct {
    using #as info: Type_Info;
    signed: bool;
}
```

## 16.3 Ways to dig into type information
See _16.1_types_in_depth.jai_:

```c++
#import "Basic";

Vector3 :: struct {
   x, y, z: float;
}

Complex :: struct { real, imag: float; }

Operating_Systems :: enum u16 #specified {
        VMS      :: 1;
        ATT_UNIX :: 2;
        WINDOWS  :: 3;
        GNU_SLASH_LINUX :: 4;
}

Direction :: enum {
    NORTH;      
    SOUTH;      
    EAST;      
    WEST;         
}

main :: () {
    // a variable can also contain a composite type:
    var := Vector3;
    print("% is of type %\n", var, type_of(var)); // => Vector3 is of type Type

    n : s32 = 5;
    a: Any = n;
    print("a's Type_Info is: %\n", << a.type); // (1) => a's Type_Info is: {INTEGER, 4}
    print("a's type is: %\n", a.type.type); // => a's type is: INTEGER
    print("a's value_pointer is: %\n", a.value_pointer); // => c3_2d9c_f844
    print("The value inside a's value_pointer is: %\n", << cast(*s32) a.value_pointer); 
    // => The value inside a's value_pointer is: 5

    if a.type.type == Type_Info_Tag.FLOAT // (2)
        print("a is a float\n");
    else if a.type.type == Type_Info_Tag.INTEGER
        print("a is an int\n"); // => a is an int

    infor := type_info(Vector3);
    print("Type of infor is: %\n", type_of(infor)); // => Type of infor is: *Type_Info_Struct
    for member: infor.members {   // (2B)
        print("%\n", member.name); // => x, y, z
    }
    print("\n");

    // type_info():
    typ1 := << type_info(float64); // (3)
    print("Type info of float64 is %\n", typ1); // => Type info of float64 is {{FLOAT, 8}}
    print("The type of typ1 is %\n", typ1.type); // => The type of typ1 is FLOAT

    typ2 := << type_info(Vector3);
    print("Type info of Vector3 is %\n", typ2); // => 
    // Type info of Vector3 is {info = {STRUCT, 12}; name = "Vector3"; 
    // specified_parameters = []; members = [{name = "x"; type = 7ff6_21db_41b0; offset_in_bytes = 0; 
    // flags = 0; notes = []; offset_into_constant_storage = 0; }, 
    // {name = "y"; type = 7ff6_21db_41b0; offset_in_bytes = 4; flags = 0; notes = []; 
    // offset_into_constant_storage = 0; }, 
    // {name = "z"; type = 7ff6_21db_41b0; offset_in_bytes = 8; flags = 0; notes = []; 
    // offset_into_constant_storage = 0; }]; status_flags = 0; nontextual_flags = 0; 
    // textual_flags = 0; polymorph_source_struct = null; initializer = null; 
    // constant_storage_size = 0; constant_storage_buffer = null; }
    print("The type of typ2 is %\n", typ2.type); // => The type of typ2 is STRUCT

    // (4) without using type_info():
    c1 := Complex.{3.14, 42.7};
    print("c1 is: % and has type %\n", c1, type_of(c1)); 
    // => c1 is: {3.14, 42.700001} and has type Complex
    TC := cast(*Type_Info) Complex;
    print("%\n", TC.type);  // => STRUCT
    if TC.type != .STRUCT then exit;
    SC := cast(*Type_Info_Struct) TC;
    print("%\n", SC.name); // => Complex
    if SC.name == "Complex" then print("This is type COMPLEX\n"); // => This is type COMPLEX
    print("c1 is of type: ");
    print("%\n", (cast(*Type_Info_Struct) type_of(c1)).name);
    // => c1 is of type: Complex

    // (5) enum_type_flags
    info := type_info(Direction); // this is a *Type_Info_Enum
    print("enum '%' is ", info.name); // => enum 'Direction' is
    if info.enum_type_flags & .SPECIFIED    print("specified.\n");
    else print("*NOT* specified.\n");       // => *NOT* specified.
    
    info = type_info(Operating_Systems); 
    print("enum '%' is ", info.name); // => enum 'Operating_Systems' is .
    if info.enum_type_flags & .SPECIFIED    print("specified.\n"); // => specified
    else print("*NOT* specified.\n");   
}
```

## 16.3.1 Cast to Any, .type and .type.type
As shown in line (1) and following, if a is a variable of type Any, then the type stored in the Any is:      
`<< a.type`, for example: {INTEGER, 4} or {STRUCT, 48}
`a.type.type` gives you the type like INTEGER or STRUCT; this is a member value from Type_Info_Tag.  
If Type1 is the type of the variable contained in the Any, then:
`<< cast(*Type1) a.value_pointer` will give you the value it contains.

We can test on a.type.type with if/else or if/case as in line (2).

## 16.3.2 The type_info() proc
There is also a `type_info()` proc: its argument must be a type and it returns a *Type_Info. Through it you can access all specific info from the specific Type_Info_ struct.
It is particularly handy when used on structs: see line 2B, where we iterate over the struct fields `members` with a for-loop (we can do that because members is an array, see ??).

It is commonly used in polymorphic procedures (see ??) to test the type of a $T argument.  
If you call `<< type_info(Type1)` on any Type1, it will give you all the info it has on that type (see line (3) and following).

The same info can be obtained by doing an explicit  
cast(*Type_Info_Struct), as shown from line (4).

## 16.3.3 Checking whether an enum is #specified
A field `enum_type_flags` in Type_Info_Enum can give you more info on the enum, for example: a flag SPECIFIED tells you whether an enum has been specified, see line (5) and following.




