---
title: C++ Coding Guidelines
parent: Contributing
---

> &#9888; IMPORTANT
>
> This is a slightly adapted version of the C++ Coding Guidelines as written down in the early 2000s. They haven't
> been updated in significant ways for a long time, and are therefore due for an overhaul.
> In particular, the recommended reading section references books that contain nowadays outdated information. 
> Furthermore, a lot of these guidelines concern themselves with formatting and naming conventions. We are in the 
> process of adopting clang-format and clang-tidy to enforce these parts automatically.

# C++ Coding Guidelines

The goal of these guidelines is to manage the complexity of C++ by describing in detail the dos and don'ts of programming in the language. These rules exist to keep the code base manageable while still allowing programmers to use C++ language features productively. Please remember that the style guidelines are not here for you to agree with every letter of them, but for all of us to follow.
When making modifications to middleware or other 3rd party software please follow a coding style that is consistent with the existing code.
When making minor modifications to CCP code that follows another coding style then stay consistent with that style. Only when you make major modifications or create new source files should you fully adhere to these guidelines.
This is not a C++ tutorial; we assume the reader is familiar with the language. For more in-depth discussions on some of the guidelines mentioned here and good C++ coding in general these books are a good read (we have a [company account for the O'Reilly Learning Portal](https://ccpgames.atlassian.net/wiki/spaces/CCP/pages/131335722/O+Reilly+Learning+Portal+formerly+Safari+Books+Online)):

- [C++ Coding Standards](https://learning.oreilly.com/library/view/c-coding-standards/0321113586/) - no-nonsense and to the point, if you read only one book, make it this one.
- [API Design for C++](https://learning.oreilly.com/library/view/api-design-for/9780123850034/)
- [Effective C++](https://learning.oreilly.com/library/view/effective-c-third/0321334876/) & [More Effective C++](https://learning.oreilly.com/library/view/more-effective-c/9780321545190/)
- [Effective STL](https://learning.oreilly.com/library/view/effective-stl/9780321545183/)
- [Modern C++ Design](https://learning.oreilly.com/library/view/modern-c-design/0201704315/)

## File headers

Start each file with a copyright notice. This applies to all source-code files, not just C++ header files. It is advised
that you configure your editor to provide appropriate templates for you.
The copyright notice has the form of `Copyright © <year of file creation> CCP ehf.`, below are a few examples.

For C++:
```c++
// Copyright © <year-of-file-creation> CCP ehf.
```
For CMake:
```cmake
# Copyright © <year-of-file-creation> CCP ehf.
```
For Python:
```python
# Copyright © <year-of-file-creation> CCP ehf.
```

Optionally you can add a comment about the general intent of the file.

## Header files

### Include guards

All header files should have `#pragma once` and `#define` guards to prevent multiple inclusion. The macro should be `<FileName>_H` or `<Project>_<FileName>_H` if the file name is too generic and likely to clash.
The `#pragma once` directive allows Visual Studio to parse the file only once and not re-open it to process the include guard every time it's referenced. This improves compile times significantly. So always put the `#pragma` first. The include guards are there to make the code portable to other compilers/platforms.
```c++
#pragma once
#ifndef Tr2Mesh_H
#define Tr2Mesh_H

// lots of awesome code

#endif // Tr2Mesh_H
```
Note that the include guard is a macro but we make an exception in how you write the macro name (it's CamelCase instead of being all uppercase). This is done so that the class name, file name and include guard all match and can be changed with a simple search-and-replace operation.

### Header file dependencies

Don't use an `#include` when a forward declaration would suffice.
Use forward declaration where possible. It reduces massive recompiles. How can we use a `class Foo` in a header file without access to its definition?
 - We can declare data members of type `Foo*` or `Foo&`.
 - We can declare (but not define) functions with arguments, and/or return values, of type `Foo`.
 - We can declare static data members of type `Foo`. This is because static data members are defined outside the class definition.
 - 
On the other hand, you must include the header file for `Foo` if your class subclasses `Foo` or has a data member of type `Foo`.

```c++
class TriRenderBatchAccumulator;
class Tr2PerObjectData;
class TriPoolAllocator;
// You can even forward declare Blue classes!
BLUE_DECLARE( TriDevice );

BLUEFACE( ITr2Renderable ) : public IRoot
{
virtual void GetNonSortedBatches( TriRenderBatchAccumulator& batches, const Tr2PerObjectData* perObjectData ) = 0;
```
Of course, implementation files (like .cpp) do require the definitions of the classes they use, and usually have to include several header files.

### Inline functions

Define functions inline only when they are small, say, 10 lines or less.
You can declare functions in a way that allows the compiler to expand them inline rather than calling them through the usual function call mechanism. If your inline function definitions are short it's fine to put them in with the function declarations. Otherwise consider putting the definitions at the end of you header file since this improves readability.

**Pros**: Inlining a function can generate more efficient object code, as long as the inlined function is small. Feel free to inline accessors and mutators, and other short, performance-critical functions.

**Cons**: Overuse of inlining can actually make programs slower. Depending on a function's size, inlining it can cause the code size to increase or decrease. Inlining a very small accessor function will usually decrease code size while inlining a very large function can dramatically increase code size. On modern processors smaller code usually runs faster due to better use of the instruction cache.

**Decision**:
A decent rule of thumb is to not inline a function if it is more than 10 lines long. Beware of destructors, which are often longer than they appear because of implicit member- and base-destructor calls!

Another useful rule of thumb: it's typically not cost effective to inline functions with loops or switch statements (unless, in the common case, the loop or switch statement is never executed).

It is important to know that functions are not always inlined even if they are declared as such; for example, virtual and recursive functions are not normally inlined. Usually recursive functions should not be inline. The main reason for making a virtual function inline is to place its definition in the class, either for convenience or to document its behavior, e.g., for accessors and mutators.

### Function parameter ordering

When creating a function, parameter order is: inputs, then outputs.

Parameters to C/C++ functions are either input to the function, output from the function, or both. Input parameters are usually values or const references, while output and input/output parameters will be non-const pointers. When ordering function parameters, put all input-only parameters before any output parameters. In particular, do not add new parameters to the end of the function just because they are new; place new input-only parameters before the output parameters.

This is not a hard-and-fast rule. Parameters that are both input and output (often classes/structs) muddy the waters, and, as always, consistency with related functions may require you to bend the rule.

### Order of includes

Use standard order for readability and to avoid hidden dependencies: your .h, C/C++ library, other libraries' .h, your project's .h.

Example: In `Foo.cpp` whose main purpose is to implement the stuff in `Foo.h`, order your includes as follows:

1. `Stdafx.h` if your project uses pre-compiled headers
2. `Foo.h`
3. C/C++ system files
4. Other libraries' .h files (external SDKs, other CCP libraries, etc )
5. Your project's .h files
6. For extra credit you can order includes alphabetically within each section. This is what it could look like:

```c++
// include "StdAfx.h" first if your project uses it
#include "Foo.h" // Preferred location.

#include <sys/types.h>
#include <unistd.h>

#include <hash_map>
#include <vector>

#include "blue/include/Blue.h"
#include "trinity/include/ITriFunction.h"

#include "FooMagic.h"
#include "FooVoodoo.h"
```
The rationale behind this is that it will ensure that `Foo.h` can stand on its own as an include file and we want that for all of our header files. The ordering for the following files is for consistency.

### Internalize external dependencies

Avoid including non-system libraries in externally exposed header files

Interfaces that are exposed to users outside of the project should endeavor to limit the inclusion of outside SDKs and libraries in the header file. Use forward declarations if possible. The rationale behind this is that otherwise the users of this interface become dependent on the external library and must set up paths to that library for inclusion. This is both a nuisance and can go horribly wrong if they for instance don't pick the right version of the external library/SDK to use for the include. Another example relates to [[BlueVectorTypes#Custom_Vector_Types|Blue Vector Types]]. So, avoid this dependency at all cost.

### Externally exposed headers

Place headers for external access into an `include` folder. These headers can not include internal headers.

The headers you can include in an externally exposed header need to meet one of the following:

1. A header that is already in the 'include' folder
2. A header that is in the 'include' folder of another project (try to avoid, but sometimes necessary)
3. A system library header

Only as a last resort can you include headers from 3rd party libraries and SDKs, always try to internalize external dependencies.

**Rationale**: Storing externally accessible headers in one place makes it explicit what is an external interface and what isn't. Knowing that .h files in 'include' can only include other .h files in the same folder prevents accidental leaking of internal data to the outside world.

### Avoid unnecessary exposure

Carefully think about what you put in a header file - the goal is to represent the external interface in the header file while not exposing internal details.

The C++ class system is unfortunately flawed in this area. The private data members and methods of a class must be present in the class declaration, and thus the header file. The [Pimpl pattern](https://en.cppreference.com/w/cpp/language/pimpl.html) can be used to address this but comes at a performance cost so it is not always a great choice. Another choice is to prefer static functions at file scope (in the .cpp file) rather than private methods in the header file. This is a neat but often overlooked solution. A third possibility is sometimes to declare and define helper classes in the implementation file. Regardless of which method works for your particular case you should always aim to expose only what's necessary.

## Naming

The most important consistency rules are those that govern naming. The style of a name immediately informs us what sort of thing the named entity is: a type, a variable, a function, a constant/macro, etc., without requiring us to search for the declaration of that entity. The pattern-matching engine in our brains relies a great deal on these naming rules.

Naming rules are in many aspects arbitrary, but we feel that consistency is more important than individual preferences in this area, so regardless of whether you find them sensible or not, the rules are the rules.

### General naming rules

Function names, variable names, and filenames should be descriptive; avoid abbreviation. Think about different shorter words that mean the same thing if the name is starting to span your screen. In general types and variables should be nouns, while functions should be "command" verbs. A ''positive'' truth statement should be preferred when naming boolean variables or functions that serve the same purpose.

Please take naming seriously and avoid creating non-descriptive but funny names. However, witty names are perfectly fine - provided they describe the subject adequately. Understand that the names you choose affect the readability and maintainability of the code so think hard before you name. Also, if the functionality or purpose has changed please update the name accordingly. While some would argue that self-documenting code is an oxymoron then misleading names are a perfect way to support that view, so don't do it.

#### Abbreviations
```c++
// Good
int dnsConnectionCount; // DNS is an accepted and well known abbreviation, fine to use
// Bad
int dnsConnCnt; // Srsly? Do your readers really know what this means? Typing isn't that tough - spell it out.
// OK
int numDnsConnections; // 'Num' is an accepted and easily understood abbreviation.
// In general prefer using 'Count' because it doesn't have to be shortened and it is postfixed rather then prefixed.
```
#### Booleans
```c++
bool IsHidden() { return m_isHidden; }
bool IsNotVisible() { return m_isHidden; } // Bad bad bad - negative truth statements are harder to understand.
bool IsVisible() { return !m_isHidden; } // Much better.
void SetHidden( bool isHidden ) { m_isHidden = isHidden; }
```

### File, type and function names

File, type, namespace and function names follow the same guideline in form, they are [CamelCase](http://en.wikipedia.org/wiki/Camelcase) starting with a capital letter. This is consistent with our Python coding guidelines. Acronyms that are longer than two letters are also written in CamelCase. This last part is consistent with the standard used in .Net and is based on research in readability. Hungarian notation is in general discouraged but we do recommend starting real interface types with a capital 'I'.

```c++
// Of course, the 'Type' suffix is just unnecessary redundancy to stress what is a type in
// this example and follows the naming rules.
// ColorConstantType.h
enum ColorConstantType {
    RED,
    BLUE,
    GREEN
};

// MyAwesomeClassType.h
class MyAwesomeClassType {
public:
    struct MyNestedStructType {
        int m_memberVariable;
    };

    void DoAwesome();
    void ThrottleIO();
    void ActivateGpsTracking();
};
```
In general try to map classes to filenames, i.e. the `class BingoBongo` is declared in the file `BingoBongo.h` and implemented in `BingoBongo.cpp`.

### Variable names

Variable names are also written in camelBack. Class member variables are prefixed with `m_` to denote scope and prevent them from clashing with local variables.

#### Local variable names
```c++
const char* names;
unsigned int namesCount;
bool isAlive;
```
#### Class variable names
```c++
class Bingo
{
private:
    std::vector<std::string> m_participantNames;
    bool m_isBingoNumberAvaiable[32];
    unsigned int m_prizeMoney;
};
```
#### Struct variable names

Structs that are plain-old-data structures should not use the member prefix, since the variables are always scoped by the containing struct. If you have structs that are equivalent to a Class with public only members then the class rules apply and an m_ prefix should be used.
```c++
struct FileHeader
{
    unsigned short version;
    unsigned char  numBytes;
    unsigned char  padding;
};

struct ClassDisguisedAsAStruct
{
    ClassDisguisedAsAStruct();
    ~ClassDisguisedAsAStruct();
    int CalculateInterestingStuff();

    int  m_secondsFromLunarEclipse;
    bool m_isGoatSacrificeDone;
};
```

### Global variable names

If you really must create a global variable consider prefixing it with `g_`. Please understand that variables with static linkage at file scope are not considered global but should be given a very specific name to reduce the chance of clashing with the global namespace during compilation of the file translation unit. A prefix of `s_`, `f_` (denotes file scope), or `t_` (thread local storage) could also do the trick.

## Constant names

Constants are named in UPPER_CASE, e.g. `COLOR_OF_MONEY`.

Non-local compile-time constants, regardless of how they are declared (#define, const, enum) are written in all uppercase. This is consistent with the way we do things in Python and the existing C++ code.

```c++
enum TriFlags {
    TRIFLAGS_SOMETHING = 1 << 0,
    TRIFLAGS_OTHER     = 1 << 1,
    TRIFLAGS_ALL       = (1 << 2 ) - 1
};

const int MY_L337_SECRET_VALUE = 2445234;

#define ANOTHER_FIXED_VALUE      10234  
// Prefer to use const and enum rather than #define

int MyFunc()
{
    const int secretCount = 10; // This is a local constant, doesn't have to follow the rules
    return MY_L337_SECRET_VALUE/secretCount;
}

class MyClass
{
public:
    enum Difficulty
    {
        EASY,
        NORMAL,
        HARD
    };
};
```

Note that enums are not scoped, i.e. many compilers will allow you to write `TriFlags::TRIFLAGS_SOMETHING` but `TRIFLAGS_SOMETHING` is the standard C++ way. This is why it's important to prefix enum constants to protect the global namespace (or scope the enums within a `namespace` or a `class`).

### Exceptions to naming rules

If you are naming something that is analogous to an existing C or C++ entity then you can follow existing naming conventions.

```c++
typedef unsigned int uint; // creating a type name which fits with the existing types
typedef unsigned char uint8_t; // creating a C99 type for a compiler that doesn't have them
class sparse_vector; // A STL-like vector requiring less storage space, named in the same fashion as std::vector.
```

## Formatting
Rules for formatting are mostly arbitrary, but a project is much easier to follow if everyone uses the same style. Individuals may not agree with every aspect of the formatting rules, and some of the rules may take some getting used to, but it is important that all project contributors follow the style rules so that they can all read and understand everyone's code easily. Realize that the formatting rules have also been chosen to simplify code merging, as our development groups grow this becomes increasingly important. [Perforce's Code in Motion article](http://www.perforce.com/beautifulcode/index.html) is a good read on this subject.

### Line length

There is no specified maximum line length. Prefer shorter lines.

Even though 24" monitors can display long lines they tend to be harder for us humans to process. Long lines can also be troublesome when doing 3-way merges or 2-way diffs. One should therefore prefer shorter lines (90 characters or less) and try to arrange the code in a "bookish" style when lines are getting long.

### Non-ASCII Characters

Non-ASCII-7 characters should be rare, use UTF-8 if you must.

If you have to put unicode text into your program consider storing it outside of the source files (resource file, external data file) or use UTF-8 encoding as a last resort.

### Spaces vs. tabs

Indent with spaces or tabs; each indentation level should be 4 spaces wide.

### Horizontal whitespace

Put spaces inside non-empty parentheses or curly braces and after a comma separator.

This is contrary to the K&R style which is used in the [[Python Coding Guidelines]]. This decision was made because most of the programmers preferred the more spacious style and it has been used in most of the newer C++ code written at CCP.
```c++
void MyFunc( int* a, float& b )
{
    if( a )
    {
        // do something
    }

    int x[] = { 0, 1 };
    for( int i = 0; i < sizeof( x )/sizeof( x[0] ); ++i )
    {
        // do more stuff
    }
 
    switch( *a )
    {
    case 1:
        b = 2;
        break;
    default:
        break;
    }
}

class Foo: public Daddy
{
public:
    // inline implementations should fit on a line
    void Reset() { m_bingo = 0; }
    Foo& operator=( const Foo& other );
};
```

There are several things to note about the spacing style outlined by this example. Notice how:

- No space before the opening parenthesis after a keyword or function but one space after
- Operator overloads are functions and adhere to the same rules about horizontal whitespace
- No space before the asterisk or ampersand but one space after


### Conditionals

Put spaces inside the parentheses and wrap the conditional statements inside scoping braces. The else keyword belongs on a new line.
Putting spaces inside the parentheses is consistent with the spacing rules used for function calls. Since many programmers at CCP program in both Python and C++ it is quite necessary to scope all conditional statements. Here is an example to illustrate this:
```python
# Snippet 1 (Python)
if a == b:
    c = 3
    return
c = 2
```
```c++
// Snippet 2 (C++)
if( a == b )
    c = 3;
    return;
c = 2;
```

The interesting part is that code snippet 1 and 2 don't work the same way. The reason being that in C++ indentation does not control scope, braces do. The simple solution is to require braces and thus prevent future mistakes:

```c++
// Modified Snippet 2 (C++)
if( a == b )
{
    c = 3;
    return;
}
c = 2;
```

Now both snippets work the same way. Another similarity is how we handle else clauses in Python and C++:

```python
# Python
if a:
    return
else:
    c = 3
```
```c++
// C++
if( a )
{
    return;
}
else
{
    c = 3;
}
```

### Loops and switch statements

Loops and switches must always use braces for the body, even when empty.

In the same way as we format if/else blocks the body of the loop or switch statement is nested inside scoping braces, even when the body is empty. The only exception to this rule are tertiary if-statements (i.e. x ? a : b ).
```c++
do
{
    c++;
} while( c < end );

while( condition )
{
    // do nothing but spin
}

switch( variable )
{
case 0:
    ...
    break;

case 1:
    {
        // this is another scope, so nest it!
        int a = GetValue();
        b = a/2;
        break;
    }

default:
    CCP_ASSERT( false );
}
```

Notice how the case statements are not indented from the body of the switch statement - this is to highlight that the scope of each case statement is the same, i.e. the body of the switch. It's only when you introduce a new scope that you should indent (as in case 1).

### Function declarations and definitions

Return type on the same line as function name, parameters on the same line if they fit.
Aim for this:
```c++
ReturnType ClassName::FunctionName( Type1 par1, Type2 par2 )
{
    // do something
}
```
If you can't fit it on one line:
```c++
ReturnType ClassName::FunctionName( Type1 par1, Type2 par2
                                    Type3 par3 )
{
    // do something
}
```
Or if you cannot even fit the first parameter:
```c++
ReturnType LongClassName::ReallyLongAndDescriptiveFunctionName(
    Type1 par1,
    Type2 par2,
    Type3 par3 )
{
    // do something
}
```
If your function is const, the keyword should be on the same line as the last parameter:
```c++
// Everything in this function signature fits on a single line
ReturnType FunctionName( Type par ) const
{
    // do something
}

// This function signature requires multiple lines, const keyword
// follows the last parameter
ReturnType ReallyLongAndDescriptiveFunctionName( Type1 par1,
                                                 Type2 par2 ) const
{
    // do something
}
```
Comment out unused parameters in function definitions:
```c++
// Always have named parameters in interfaces.
class Shape
{
public:
    virtual void Rotate( double radians ) = 0;
}

// Always have named parameters in the declaration.
class Circle : public Shape
{
public:
    virtual void Rotate( double radians );
}

// Comment out unused named parameters in definitions.
void Circle::Rotate( double /*radians*/ )
{
    // do something
}
```

Avoid `int foo( void )`, C-style code. Drop the "void".

### Function calls

On one line if it fits; otherwise, wrap at first argument.

Function calls have the following format:
```c++
bool retval = DoSomething( argument1, argument2, argument3 );
```
If the arguments do not all fit on one line, they should be broken up onto multiple lines, with each subsequent line aligned with the first argument:
```c++
bool retval = DoSomething( aVeryVeryVeryVeryLongArgument1,
                           argument2, argument3 );
```
If the function has many arguments, consider having one per line if this makes the code more readable:
```c++
bool retval = DoSomething( argument1,
                           argument2,
                           argument3,
                           argument4 );
```
If the function signature is so long that it cannot fit within the maximum line length, you may place all arguments on subsequent lines:
```c++
if(...)
{
    // ...
    // ...
    if (...)
    {
        DoSomethingThatRequiresALongFunctionName(
            very_long_argument1,  // 4 space indent
            argument2,
            argument3,
            argument4 );
    }
}
```

### Preprocessor directives

Preprocessor directives should not be indented but should instead start at the beginning of the line.
Indentation of code inside conditionally compiled blocks must follow the regular indentation rules. In order for the preprocessor directives that control the conditional compilation to still be clearly distinguishable we start them at the beginning of the line:
```c++
if( filePath )
{
#if !defined(NDEBUG)
    VerifyFilePath( filePath );
#endif
    OpenFile( filePath );
}
```

### Namespace and class formatting

Contents of namespaces are not indented. The public, protected and private keywords in classes are not indented.
Namespaces do not add an extra level of indentation; For example:
```c++
namespace {

void Bingo() // Correct, no extra indentation
{
// do something
}

} // anonymous namespace ends
```
The public, protected, and private keywords do not add an extra level of indentation:
```c++
class MyClass
{
public:
    void MakeAwesomeSauce();
private:
    bool m_isSauceAwesome;
};
```

### Pointer and Reference Expressions

No space before asterisk or ampersand on type declaration.
The reference or pointer trait is a characteristic of the type, not the name of the variable.
```c++
// Do
int& b = a;
int* c = &a;
*c = 3;

// Don't!
int &b = a;
int *c = & a;
* c = 3;

// Beware of this, though:
int* c, d; // declares c as pointer to int but d as int
// This is better
int* c;
int* d;  
// Sure, it's another line but the code is also easier to merge because a subsequent
// type change for 'd' now only affects the type declaration line for 'd'
```

### Type casting
C++ style casting is safer than c-style casting and easier to search for.
For example:
```c++
float a = 2.5;

// DO
int b = static_cast<int>( a );

// DON'T
int b = (int)a;
```

### Boolean expressions

Boolean expressions that are too long to fit on a line should be broken up consistently.
For example:
```c++
if( thisOneThing > thatOtherThing &&
    aThirdThing == aForthThing ||
    yetAnotherThing )
{
    //...
}

// If indentation allows for it then consider starting each line with the and/or operator:
void MyFunction()
{
    if( thisOneThing > thatOtherThing
     && aThirdThing == aForthThing
     || yetAnotherThing )
    {
        //...
    }
}
```
Feel free to add parentheses judiciously, because they will increase readability when used appropriately.

### Constructor initializer lists

Constructor initializer lists can be all on one line or with subsequent lines indented - in declaration order.

There are two acceptable formats for initializer lists:
```c++
// When it all fits on one line:
MyClass::MyClass( int var ) : m_memberVar1( var ), m_memberVar2( nullptr ) {}

// When it requires multiple lines:
MyClass::MyClass( int var ) :
    m_memberVar1( var ),
    m_memberVar2( nullptr )
{
    // ....
}
```
Please list the initializers in the declaration order. This is because C++ executes the initializers in declaration order, not in the order specified in the definition. By keeping them the same there's less confusion.

### Avoid redundant initializers

Avoid doing this:
```c++
Foo::Foo()
: bar1()
, bar2()
{}
```
bar1 and bar2 will already be default constructed without these unneeded statements. What's more, this code suggests that bar1 will be constructed before bar2, but whether that is the case or not depends on the declaration order of bar1 and bar2 in the class itself, see previous item.

## Classes

### Declaration order

Use the specified order of declarations within a class: public before private, methods before member variables, etc.

Your class definition should start with its public section, followed by its protected section and then its private section. Use only one section of each type and if any of these sections are empty, omit them.

Do not put large method definitions inline in the class definition. Usually, only trivial or performance-critical, and very short, methods may be defined inline.

Group interface method declarations that your class inherits and implements together and identify them:
```c++
/////////////////////////////////////////////////////////////////////////////
// ITriDeviceResource
bool PrepareResources();
void ReleaseResources( TriStorage s );
```

### Limit work in constructors

In general, constructors should merely set member variables to their initial values (if those variables don't have default constructors or the defaults are not desired). Any complex initialization should go in an explicit Initialize() or Init() method.

The problems with doing too much work in constructors are:
1. There is no easy way for the constructor to signal errors, short of using exceptions (which we avoid at all cost).
2. If the work fails, we now have an object whose initialization code failed, so it may be in an indeterminate state.
3. If the work calls virtual functions, these calls will not get dispatched to any subclass implementations. Future modification to your class can quietly introduce this problem even if your class did not have a subclass version of the particular method originally.
4. If someone creates a static variable at file scope of this type, the constructor code will be called before main(), possibly breaking some assumptions about program state in the constructor code.

### Explicit constructors

Use the C++ keyword explicit for constructors with one argument.

Normally, if a constructor takes one argument, it can be used as a conversion. For instance, if you define Foo::Foo( int i ) and then pass an integer to a function that expects a Foo, the constructor will be called to convert the argument into a Foo and pass the Foo to your function for you. This can be convenient but it is also a potential pitfall. Declaring a constructor explicit prevents it from being invoked implicitly as a conversion. Exceptions to this rule are copy constructors and classes that are intended to be transparent wrappers around other classes.

### Copy constructors

Provide a copy constructor and assignment operator only when necessary. Otherwise, disable them.

The copy constructor and assignment operator are used to create copies of objects. The copy constructor is implicitly invoked by the compiler in some situations, e.g. passing objects by value.

Copy constructors make it easy to copy objects. STL containers require that all contents be copyable and assignable. Copy constructors can be more efficient than CopyFrom()-style workarounds because they combine construction with copying, the compiler can elide them in some contexts, and they make it easier to avoid heap allocation.

Implicit copying of objects in C++ is a rich source of bugs and of performance problems. It also reduces readability, as it becomes hard to track which objects are being passed around by value as opposed to by reference, and therefore where changes to an object are reflected.

Few classes need to be copyable. Most should have neither a copy constructor nor an assignment operator. In many situations, a pointer or reference will work just as well as a copied value, with better performance. For example, you can pass function parameters by reference or pointer instead of by value, and you can store pointers rather than objects in an STL container.

If your class needs to be copyable, prefer providing a copy method, such as CopyFrom() or Clone(), rather than a copy constructor, because such methods cannot be invoked implicitly. If a copy method is insufficient in your situation (e.g. for performance reasons, or because your class needs to be stored by value in an STL container), provide both a copy constructor and assignment operator.

If your class does not need a copy constructor or assignment operator, you must explicitly disable them. To do so, add dummy declarations for the copy constructor and assignment operator in the private: section of your class, but do not provide any corresponding definition (so that any attempt to use them results in a link error).

## Code Commenting

We are using [http://www.doc-o-matic.com/ Doc-O-Matic] to auto-generate HTML documentation for the Trinity and Blue projects. We adhere to the basic Doc-O-Matic commenting standard (described below). As always, when making minor edits to a .h or .cpp file, it is not necessary to attempt to convert the entire file comments to Doc-O-Matic standard. But if you are making substantial changes to a class, or adding a new class, make sure you follow the commenting standards. Of course, any new member functions or data members should be commented accordingly.

### Commenting class declarations
Provide a comment block before the class declaration containing a mandatory Description section.
The Description section is required. Other sections, like SeeAlso and Summary are optional.
```c++
Example:
// -------------------------------------------------------------
// Description:
//   MyClass is a class that does stuff.  It is managed by the
//   MyClassManager class, and contains a list of MyThings.
// SeeAlso:
//   MyClassManager, MyThing
// -------------------------------------------------------------
class MyClass
{
//...
};
```

### Commenting class data members

If the meaning and usage of a data member is not obvious, please provide a single-line comment immediately before each public, protected, and private data member succinctly describing its role.

No section tags like Description are required. If the comment length would violate the line-length guideline, break it into multiple lines. Example:
```c++
Example:
class MyClass
{
public:
    //...

private:
    // Matrix describing the position, orientation, and scaling of
    // the object in world space
    Matrix m_transformMatrix;
    // Hides/unhides the object on the screen
    bool m_isHidden;
};
```

### Commenting class member functions
You may optionally provide a single-line comment immediately before each public, protected, and private member function declaration describing its function or behavior. Provide a detailed comment block immediately before the function implementation in the .cpp file.
In the implementation file, the Description section is required. If the function takes 1 or more arguments, then the Arguments section is required. If the function returns a value, then the Return Value section is required. You may also include See Also and Summary sections, but these sections are optional.
Example header file (class declaration comment omitted for clarity):
```c++
// MyClass.h

class MyClass
{
public:
    // Calculates the distance of the object from the camera.
    float CalcDistanceFromCamera( const Vector3& cameraPos );
};
```
Example source file:
```c++
// MyClass.cpp
#include "MyClass.h"

// -------------------------------------------------------------
// Description:
//   Calculates the distance of the object from the camera, in
//   world coordinates.  Note that this calculation is approximate
//   since it uses the object's bounding box, not the actual
//   mesh data.
// Arguments:
//   cameraPos - position of the camera in world space
// Return Value:
//   Distance of the object from camera in world units (meters)
// SeeAlso:
//   CalcBoundingBox
// -------------------------------------------------------------
float MyClass::CalcDistanceFromCamera( const Vector3& cameraPos )
{
    // ...
}
```
If the return value is a boolean or enumeration, you should explicitly document the meaning of the return value like this:
```c++
// Return Value:
//   true If memory allocation succeeded
//   false if memory allocation failed
```
Enumeration example:
```c++
// Return Value:
//   D3DFMT_D24X8 If no stencil buffer was requested
//   D3DFMT_D24S8 If a stencil buffer was requested
//   D3DFMT_UNKNOWN If something bad happened
```

### Commenting non-member functions

Follow the same guidelines as for member functions.

### Commenting for Python users

Follow the [[Blue Docstring Guidelines]]when commenting C++ functionality exposed to Python (most often through [[Blue]]).
### Miscellaneous Notes

You can easily add external links to resources such as the Core Wiki from within Doc-O-Matic comment blocks.
To add a single link, use an extlink tag on a new line within the Description section like this:
```c++
// -------------------------------------------------------------
// Description
//   Some highly informative text about MyClass.
//   <extlink http://core/wiki/MyClass>MyClass on CoreWiki</extlink>
//
```
To add multiple links, use a Link List section in the comment block like this:
```c++
// -------------------------------------------------------------
// Description
//   Some highly informative text about MyClass.
// Link List
//   <extlink http://core/wiki/MyClass>MyClass on CoreWiki</extlink>
//   <extlink http://core/wiki/MyRelatedClass>MyRelatedClass on CoreWiki</extlink>
//
```
To exclude a code block from the built HTML documentation, surround the block with the appropriate ignore tags.
Example:
```c++
//DOM-IGNORE-BEGIN
void MyFunctionToExclude( const char* c )
{
    // Do something...
}
//DOM-IGNORE-END
```

Don't write comments that are a hidden copy-paste of the code. It's a waste of time, and it's extra bad if the code is updated and the comment isn't, or if the code gets moved around and the comment doesn't follow along.

```c++
// ----------------------------
// Good:
//
// Make sure we receive updates
BigSystem::Register( this );
// ----------------------------

// ----------------------------
// Bad:
//
// Register with BigSystem
BigSystem::Register( this );
// ----------------------------

// ----------------------------
// Bad:
//
// Register with BigSystem
DoSomethingTotallyDifferent();
// ----------------------------

// ----------------------------
// Bad:
//
// Register with BigSystem
DoSomethingTotallyDifferent();

DoAnotherThing();

BigSystem::Register( this );
// ----------------------------
```

## Random useful tidbits

### Avoid throwing C++ exceptions

Be sure to wrap catch statements around 3rd party code that can throw exceptions. For your own code, stay away from this C++ feature.
If your project is very small, ''and'' all code paths can be guaranteed to catch your exception, ''and'' clean-up is handled by RAII for '''all resources acquired''' (otherwise every throw will leak memory), ''and'' you are very familiar with the C++ implications that exceptions bring with them, ''then'' you can use exceptions. So, most likely you can't use exceptions. Instead use CCP_LOG to report errors, use error codes and throw Python exceptions (by returning nullptr from [[Blue]]wrappers, ideally after calling PyErr_SetString).

### Don't mix references and pointers for similar arguments

References are very useful for having the same effect as a pointer, without the worry of checking for NULL values. However, avoid mixing pointers and references when used for the same purpose, as this can create confusion.

An example of inconsistency creating a confusing function signature:
```c++
void DeriveFrustum(
const Matrix* view,
const Vector3* campos,
const Matrix* projection,
const TriViewport& viewport );

frustum.DeriveFrustum(
&Tr2Renderer::GetViewTransform(),
&Tr2Renderer::GetViewPosition(),
&Tr2Renderer::GetProjectionTransform(),
Tr2Renderer::GetViewport() );
```
Please note that it's still perfectly valid to use const reference for read-only arguments and pointers for output arguments. These are different arguments in purpose and this separation can actually lead to increased code clarity. Example:
```c++
// usage: CalcBoundingBox( geo, &min, &max );
void CalcBoundingBox( const Geometry& geo, Vector3* minPoint, Vector3* maxPoint );
```
### Do not add function parameters of boolean

Two major reasons for this guideline: readability and extensibility. Functions like this: DisplayText( "This is bold but not italic", true, false ) stink.
The example function above is not only hard to read and understand (what do the boolean parameters mean) but if we were to add an underline option we would tack on yet another boolean parameter. That's the extensibility problem. If we switch to enumerations the function could read like this:
```c++
DisplayText( "This is bold, underlined, but not italic", TextFormat::BOLD | TextFormat::UNDERLINE );
```
which is both more readable and easier to extend in the future. More discussion here. You should also be careful to not fall into the trap of extending an existing function with new functionality and having that new functionality switchable with a boolean parameter. In those cases it is better to expose two public functions with different descriptive names (they can still share as much common code as you want, use a private common function or have one call the other):
Please note that this guideline does not pertain to boolean accessor funtions, i.e. this is fine:
```c++
// turn on the display
SetDisplayEnabled( true );
// now turn it off
SetDisplayEnabled( false );
// now check
if( IsDisplayEnabled() )
{
    // do something cool
}
```
As you can see the code above is readable and there is no extensibility issue either because these are accessors.
Now, there may be valid exceptions where it makes sense to violate this rule. However, you should at all cost refrain from creating Swiss-Army-Knife functions, i.e. functions that do many radically different things based on boolean input parameters.

### Use NDEBUG, not _DEBUG

The C standard guarantees that NDEBUG is set in release and not in debug targets. Don't rely on preprocessor defines like _DEBUG or DEBUG being set by the compiler when conditionally compiling for release or debug targets.
Please note that you are free to declare whatever preprocessor define that makes more sense for you based on the setting of NDEBUG. This guideline only states that you cannot rely on _DEBUG in a vanilla configuration of a project, but you can can rely on NDEBUG being properly set.

### Prefer light orthogonal interfaces

- Small Orthogonal Interfaces, not massive class inheritance.
- Interface class names should start with an 'I'
- Interface classes should only declare (and possibly define) methods, never hold data
- Interfaces that are exposed and used outside of a DLL should be stored in an 'include' folder - these kinds of interfaces must be careful to expose as little as possible of the DLL internals (SDK includes, etc). Use forward declarations as much as possible.

### Public / Protected / Private

Make variables private if possible, protected if not, public as a last resort.

### STL, C++0x

- Use `nullptr` instead of `NULL` for much improved type-safety.
- Prefer to use `empty()` rather than a zero `size()` comparison.
- For some container `v`, prefer `if( v.empty() )` over `if( !v.size() )`  
  First of all, it's more readable and explicit about what you want to know. Secondly, it could be more efficient: if `v` is a map or a list, you may pay the cost of a traversal to compute the size, only to throw away the answer.
- If you don't care about element order, the new `unordered_map`, `unordered_set` are faster.
- If you use `auto` to iterate over containers, use `cbegin`/`cend` and friends if you want a `const_iterator`. They exist on `BlueList` too.
- Don't use `auto` for absolutely everything and anything. Make sure you understand the type deduction rules for corner cases. If you expect a reference, pointer, const qualifiers, etc add them explicitly.
```c++
foo& bar() { ... }
auto oops = bar(); // not a reference!
auto& ok = bar(); // Ok.
```

- Lambda functions make `ON_BLOCK_EXIT` a whole lot more convenient, and often easier to read. Use it.
Before:
```c++
ON_BLOCK_EXIT( &TID3DVertexBuffer::Unlock, m_pVertexBuffer.p );
```
After:
```c++
ON_BLOCK_EXIT( [&]{ m_pVertexBuffer->Unlock(); } );
```
Notice how you can omit empty parentheses after [&] for slightly less visual noise.

- [MSVC11] The previous auto guideline is especially true in ranged for loops.
If the loop does something to the variable, use `auto&` :
```c++
for( auto& x : vec )
{
    x *= 3;
}
```
- If the loop does not change the container contents, use const auto or const auto&:
```c++
for( const auto x : vec )
{
    cout << x;
}
```
- In short, you probably do not want to do this:
```c++
for( auto x : vec )
{
    // ...
}
```

- Use the new smart pointers, especially `unique_ptr`.
```c++
// pretty good:
char* data = new char[m_bitmapWidth * m_bitmapHeight * 4];
ON_BLOCK_EXIT( [&]{ delete[] data; } );

// even better:
std::unique_ptr<char[]> data( new char[m_bitmapWidth * m_bitmapHeight * 4] );

// ... or in this case ([[Memory_Tracking#Standard_containers|details here]]):
CcpMallocBuffer data( "myTrackingName", m_bitmapWidth * m_bitmapHeight * 4 );
```

## Boost Python vs Blue
Please don't cross the streams.

## GOTO

You run the risk of getting shot. All GOTO shenanigans, could be done much more robustly using [[Scope Guard]].

## Evaluation SDKs

Use the `NO_EVALS` compiler define so that we don't have to ship them out in releases.

---- This marks the end of the original C++ coding styleguide, following are the Platform Agnostic Guidelines ----

## General Recommendations

### Naming files and folders

Use `lower_case` when naming files and folders, e.g. all lower-case, if need be separate words with an underscore. Doing so avoids remembering exact casing on file systems that are case sensitive (e.g. most posix-based file systems). It also avoids surprise build failures on other platforms when someone included `"SomeInclude.h"` as `"someinclude.h"`.

### Interfacing with platform specific code from C++

No matter the amount of abstractions we put into place, at one point the need arises to interface with the underlying platform. Here are a couple of guidelines and tips for doing so. Generally we will want to keep the places that do need to interface with the platform to an absolute minimum. In our codebase this will most likely mean the `CcpCore`, `CcpMath`, `Exefile` and `Trinity` projects, which provide abstractions for common functionality (`CcpCore`, `CcpMath`), executable file (`Exefile`) and rendering (`Trinity`).
Discovering predefined macros for platforms

- `msvc`: [See the official documentation](https://docs.microsoft.com/en-us/cpp/preprocessor/predefined-macros?view=vs-2019)
- `clang`: run the command `clang -dM -E -x c /dev/null` in your terminal
- `gcc`: run the command `clang -dM -E -x c /dev/null` in your terminal

### Guarding platform specific code using preprocessor macros

We recommend using `#if <platform macro> #elif <platform macro> #else #error Unsupported platform #endif` to guard platform specific code. The rationale here is that we want the compiler to complain very loud about missing implementations on unsupported platforms, and it is easier to change the value of a macro than undefining it. If you are stubbing out an implementation then add  `#pragma message "WARNING: ..."` line - it helps a lot to get reminded about these stubs during the build process.
See the following example for what this could look like:
#### Platform specific code using preprocessor macros

```c++
#if _WIN32
#pragma message("WARNING: Windows uses a stub implementation!")
#elif __APPLE__
#pragma message("WARNING: macOS uses a stub implementation!")
#elif __linux__
#pragma message("WARNING: Linux uses a stub implementation!")
#else
#error Unsupported platform!
#endif
```

### Guarding architecture specific code using preprocessor macros

This should be done in the same way as we guard platform specific code, e.g.:

#### Architecture specific code using preprocessor macros

```c++
// These are MSVC specific architecture preprocessor macros
#if _M_ARM64
#pragma message("ARM64 code should be written here")
#elif _M_X64
#pragma message("X64 code")
#else
#error Unsupported architecture!
#endif
```
Please consult your compiler as outlined above for a list of useful identifiers for now as we do not yet have any predefined macros for this ourselves.

### Platform specific code in CMake

CMake provides a couple of variables that describe the system. For example, we can use the variable `CMAKE_SYSTEM` to identify which operating system we are building for. The variable `CMAKE_HOST_SYSTEM` contains information which operating system we are building on.

The variable `CMAKE_CXX_COMPILER_ID` can be used to distinguish between different compilers. The variables `CMAKE_VS_PLATFORM_TOOLSET` and `CMAKE_XCODE_PLATFORM_TOOLSET` can be used to control the platform toolset used by the Visual Studio and XCode generators.

One can use [generator expressions](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html) to add compiler-specific options to a target. These can be difficult to read, however.
#### Example: Enabling warning levels based on compiler

```cmake
target_compile_options(example PRIVATE
    $<$<CMAKE_CXX_COMPILER_ID:Clang,AppleClang,GNU>:-Wall -Werror>
    $<$<CMAKE_CXX_COMPILER_ID:MSVC>:/Wall /WX>
)
```
