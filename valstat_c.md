<h1>valstat - C valstat</h1><!-- omit in toc -->
 
| &nbsp;           | &nbsp;                                                |
| ---------------- | ----------------------------------------------------- |
| Document Number: |                                            |
| Date             | 2021-03-13                                            |
| Audience          | WG14       |
| Author         | Dusan B. Jovanovic ( [dbj@dbj.org](mailto:dbj@dbj.org) ) |


There are two ways of constructing a software design: One way is to make it so simple that there are obviously no deficiencies, and the other way is to make it so complicated that there are no obvious deficiencies. -- [C.A.R. Hoare](https://en.wikiquote.org/wiki/C._A._R._Hoare)

## Table of Contents<!-- omit in toc -->

- [1. Abstract](#1-abstract)
- [2. From error handling to interoperability](#2-from-error-handling-to-interoperability)
- [3. valstat protocol C definition](#3-valstat-protocol-c-definition)
  - [3.2. Type requirements](#32-type-requirements)
  - [3.4. bare-bones valstat](#34-bare-bones-valstat)
- [4. Usage](#4-usage)
- [5. Conclusions](#5-conclusions)
- [6. References](#6-references)
- [7. Appendix A](#7-appendix-a)
  - [7.1. valstat as a solution for known and difficult problems](#71-valstat-as-a-solution-for-known-and-difficult-problems)
  - [8.2. Fully functional valstat type](#82-fully-functional-valstat-type)
- [8. Appendix: Requirements Common Across Domains](#8-appendix-requirements-common-across-domains)
  - [8.2. Interoperability](#82-interoperability)
<h2>&nbsp;</h2>

## Revision history<!-- omit in toc -->

R0: Created

<!-- div class="page"/ -->

## 1. Abstract

This is a proposal about logical, feasible, lightweight and effective handling of information returned from C functions, based on the [valstat protocol](https://github.com/DBJDBJ/valstat).

valstat is not yet another error handling idiom. Please take a slight and quick detour to read that document first.

<!-- Also this paper proposes some possible solutions, to some of deeply rooted ISO C++ issues [[3](#ref3)][[15](#ref15)].  -->

Implemented in standard C, this would be a tiny standard C library citizen without any language change required. Although not even that is required to adopt implement and use valstat protocol in software projects, coded in standard C.

For standard C formal definition please see [[STDC]](#REF_STDC)

<span id="motivation" />

## 2. From error handling to interoperability

C is the language that every runtime understands.  But there is no standard for the full information passing out of C functions to other languages run-times.

valstat C is that missing link.

As of today, there are more than few, **error** handling paradigms, idioms and return types used in C projects. Accumulated through decades, from ancient to contemporary. Together they have inevitably contributed to a rising technical debt present inside C coded projects. 

valstat C can also offer standard protocol for sorting out interoperable information passing between "foreign" C libraries.

In order to achieve the required wide scope of the valstat protocol, implementation has to be simple. valstat actual programming language shape has to be completely problem domain or context free. Put simply the C implementation must not influence or dictate the usage beside supporting the protocol. 

## 3. valstat protocol C definition

Recap

valstat protocol defines information as made of state and data. valstat structure is made by responder and returned as an full information carrier. it is made of two fields: value and status.
```
structure: valstat
   field: value
   field: status
```
Where valstat protocol field has two occupancy states
```
    field_state ::= empty | occupied
```

The valstat caller, has the opportunity to decode (aka capture) one of the four states, carried over from a call responder. 

> NOTE: That is wider scope than simple error handling

Information carried over by valstat structure is handled ny callers, in two steps:

1. Step One
   - decode the state for step two
2. Step Two
   - use the data 

### 3.2. Type requirements

Both value and status field types, should offer a simple mechanism that reveals their occupancy state. 

Step One decoding in C should look as simple as this:
```cpp
// both occupied
  if (   value &&  status ) { /* state: info */ }
// occupied and empty  
  if (   value && !status )  { /* state: ok   */ }
// empty and occupied  
  if ( !value  && status )  { /* state: error*/ }
// empty and empty  
  if ( !value && !status )  { /* state: empty*/ }
```
What is the meaning of "empty" for a particular  type, and what is not, depends on the context. In some contexts pointer can be "empty" if it is NULL. Or zero (0) can have the meaning of empty in some other context.

Reminder: in return decoding step one, logically we use only states of the fields.

### 3.4. bare-bones valstat 

It is quite ok and enough to be using fundamental types for both value and status fields. We can implement the "field" paradigm by using just C fundamental types.

Example

```cpp
   // 
   typedef struct {
      int value;
      int status;
   } valstat_int_int ;
```
Both value and status fields in here are just integers. Yet we can simply and comfortably represent four valstat states.
```cpp   
valstat_int_int vii_ok    = { 42, 0 }; // OK state
valstat_int_int vii_error = { 0,  error_code }; // ERROR state
valstat_int_int vii_info  = { 0xFF, status_code }; // INFO state
valstat_int_int vii_empty = { 0, 0 }; // EMPTY state
```
It all depends on the context. That is still valstat protocol in action. In some situations valstat field types can be two simple integers. 

Above is rather important valstat ability to be transparently adopted for various projects. That solution is not using any special types and can work under extremely strict runtime conditions. 

<!-- div class="page"/ -->

## 4. Usage

<!-- ### 4.1. Callers point of view

Recap: my::valstat struct instance carries (out of the functions) information: state and payload to be utilized by callers. How and why (or why not) is the valstat state decoding algorithm shaped, that completely depends on the project, the API logic and many other requirements dictated by adopters architects and developers. 

Example bellow is used by valstat adopters operating on some database. In this illustration, adopters use the valstat to pass back (to the caller) full information (state + data), obtained after the database field fetching operation. Again, please notice there is no 'special' over-elaborated return type required. That is a good thing. valstat is a protocol, there is no complex C++ type, just clean and repeatable idioms of two step returns handling.

 ```cpp
 // declaration of a valstat emitting function
 template<typename T>
// we use my::valstat type from above
// `my::stat` is 'code' from some internal code/message mechanism.
   my::valstat<T, my::stat > 
   full_field_info
   (database::row /*row_*/ , std::string_view /* field_name */ ) 
// valstat protocol naturally allows no exception throwing   
    ;
```
Primary objective is enabling callers comprehension of a full information passed out of the function, state and data. Full returns, not just error handling. 
```cpp
// full return handling after 
// the attempted field content retrieval
auto [ value, status ] = full_field_info<int>( db_row, field_name ) ;
```
When designing a solution, adopters have decided they will utilise all four valstat states. Calling code is capturing all.
```cpp
// step one
// capturing: info 
if (   value &&   status )  { 
   // step two
   std::cout << "\nSpecial value found: " << *value ;
   // *status type is my::stat
   std::cout << "\nStatus is: " << my::status_message(*status) ;
  }

// capturing: ok 
if (   value && ! status )  { 
   std::cout << "\nOK: Retrieved value: " << *value ;
  }

// capturing: error 
if ( ! value &&   status )  { 
   // in this design status contains an error code
   std::cout << "\nRead error: " <<  my::status_message(*status) ;
  }

// capturing: empty 
if ( ! value && ! status )  { 
   // empty feild is not an error
   std::cout << "\nField is empty." ;
  }

 ```
Please do note, using the same paradigm it is almost trivial to imagine that same code above, in e.g. JavaScript, calling the module written in C++ returning valstat protocol structure that JavaScript will understand. 

Let us emphasize: Not all possible states need to be captured by the caller each and every time. It entirely depends on the API design, on the logic of the calling site, on application requirements and such.

### 4.2. Responder: the API point of view

Requirements permitting, API implementers are free to choose if they will use and return them all, one,two or three valstat states. 

```cpp
// API implementation using valstat protocol
template<typename T>
my::valstat<T, my::stat > 
full_field_info
(database::row row_, std::string_view field_name ) 
// platform requirements do not allow
// throwing exceptions
             
{
   // sanity check
   if ( field_name.size() < 1) 
    // return ERROR state
      return { {}, my::stat::name_empty };      

   // using some hypothetical database API
   // where row is made of fields
   database::field_descriptor field = row_.fetch( field_name ) ;
 
    // error can be anything not
    // just database related
    if ( field.in_error() ) 
    // return ERROR valstat
    // status is some internal code of my::stat type
      return { {}, my::stat::db_api_error( field.error() ) };      

    // empty field is not an error
    // return an EMPTY valstat structure
    if ( field.is_empty() ) 
      return { {}, {} };      

   // db type will have to be cast into the type T
   // type T handles value semantics
   T field_value{} ; 
   // try getting the value from a database field
   // and casting it into T
   if ( false == field.data( field_value ) )
   // failed, return ERROR state 
      return { {},  my::stat::type_cast_failed( field.error() ) }; 

 // solving business requirement
 // API contract requires signalling if 'special' value is found
  if ( special_value( field_value ) )       
  // return INFO state and both fields populated
   return { field_value, my::stat::special_value }; 

// value is obtained and ready
// status field is empty
// OK state signalled back 
   return { field_value, {} }; 
}
```
Basically function returning the valstat state + data, is simply returning two fields structure. With all the advantages and disadvantages imposed by the core language rules. Any kind of home grown but functional valstat type will work in there too. As long as callers can capture the states and data in two steps by using the two fields returned.

Using thread safe abstractions, or asynchronous processing is also not stopping the adopters to return the metastates from their API's.

<div class="page"/ -->

## 5. Conclusions

*Fundamentally, the burden of proof is on the proposers.* — B. Stroustrup, [[11]](#ref11)

<!-- Hopefully proving the evolution of error code handling into returns handling does not need much convincing. There are many real returns handling situations where the valstat paradigm can be successfully used. As an common call returns handling paradigm, valstat requires to be ubiquitously adopted to become truly an instrumental to widespread interoperability. From micro to macro levels. From inside the code to inter component calls. From inside an project to inter team situations. -->

"valstat" protocol is multilingual in nature. Thus adopters from any imperative language are free to implement it in any way they wish too. The key protocol benefit is: interoperability. 

Using the same protocol implementation it is feasible to develop standard C++ code using standard library, but in restricted environments. Author is certain readership knows quite well why is that situation considered unresolved in the domain of ISO C++. 

<!-- There is no need for yet another [tractate](https://www.merriam-webster.com/dictionary/tractate), in the form of proposal to try and explain the background.  -->

Authors primary aim is to propagate widespread adoption of this paradigm. As shown valstat protocol implemented in C++ is more than just solving the "error-signalling problem"[[11](#ref11)]. It is an paradigm shift, instrumental in solving the often hard and orthogonal set of platform requirements described in the [motivation section](#2-motivation).

valstat protocol and C++ definition, while imposing extremely little on adopters is leaving the non-adopters to "proceed as before".

Obstacles to paradigm adoption are far from just technical. But here is at least an immediately usable attempt to chart the way out.

----------------------

<!-- <div class="page"/> -->

## 6. References

<a id="REF_STDC">[STDC]</a> 
ISO/IEC 9899:2018 -- Information technology — Programming languages — C, https://www.iso.org/standard/74528.html

- <a id="ref_modernc">[MODERNC]</a> Modern C, Jens Gustedt, https://modernc.gforge.inria.fr

- <a id="ref_empty">[EMPTY]</a> "Your Dictionary" **Definition of empty**,  https://www.yourdictionary.com/empty


<!-- div class="page"/ -->

## 7. Appendix A

*To me, one of the hallmarks of good programming is that the code looks so simple that you are tempted to dismiss the skill of the author. Writing good clean understandable code is hard work whatever language you are using -- Francis Glassborow*

Value of the programming paradigm is best understood by seeing the code using it. The more the merrier. Here are a few more simple examples illustrating the valstat protocol and implementations applicability.

### 7.1. valstat as a solution for known and difficult problems

An perhaps very elegant solution to the "index out of bounds" problem. Using [my::valstat](#41-myvalstat) as already defined above. 

```cpp
// inside some sequence like container
// note how we use pre existing std type
// for the status field
 my::valstat< T , std::errc >
     operator [] ( size_t idx_ ) 
    {
        if ( ! ( idx_ < size_ ) )
        /* ERROR state + data */
        return { {}, my::errc::invalid_argument };
        /* OK state + data */
        return { data_[idx_] , {} };
    }
```
That usage of my::valstat alone resolves few difficult and well known design issues.
```cpp
auto [ value, status ] = my_vector[42] ;

// first step: check the states
if ( value  )  { /* second step: we are here just if state is OK     */ }
if ( status )  { /* second step: we are here just if state is ERROR  */ }
```
No exceptions, no `assert()` and no `exit()`. 

<span id="home_grown_crt_valstat" />

### 8.2. Fully functional valstat type

Let us assume we need to write a layer of safe proxies to some C run time (CRT) functions. We might use various valstat structs where the status field is actually the errno value. We can decalre them by hand or using simple macro

```cpp
// 
typedef int errno_field_type ;

#declare ERRNO_VALSTAT( T )      \
typedef struct {                 \
	T   value;                    \    
	errno_field_type  status;     \
} T##_errno_valstat ;         
```
Usage
```cpp
ERRNO_VALSTAT(int) ;
```
Generates
```cpp
typedef struct {
	int   value;
	errno_field_type  status;
} int_errno_valstat ;

```
Very common `char *` variant
```cpp
// generates char_ptr_errno_valstat
typedef char * char_ptr ;
ERRNO_VALSTAT(char_ptr) ;
```
That valstat variant might be used in a myriad of API's, internally facing the CRT, or CRT like API's. All returning the information in the shape of state + value, carried by the very few declarations created by that one macro . Examples:

```cpp
// assume some proxy functions to the CRT
// 
char_ptr_errno_valstat safe_read_line ( FILE * )  ;

double_ptr_errno_valstat safe_sqrt( double * ) ;

char_ptr_errno_valstat safe_strdup( const char * )  ;
```
The caller using any of the above imagined crt proxy functions, will follow the same valstat two step state capturing idiom returns processing.

Let's see how modern_fopen(), might be actually implemented.

```cpp
// generates f_ptr_errno_valstat
typedef FILE * f_ptr ;
ERRNO_VALSTAT(f_ptr) ;
// 
inline f_ptr_errno_valstat 
  modern_fopen(const char* name_, const char* mode_) 
{
    // the ERROR valstat is returned on bad arguments
    if (NULL == name_)
        return (f_ptr_errno_valstat){ NULL ,  EINVAL }; 

    if (NULL == mode_)
        return (f_ptr_errno_valstat){ NULL ,  EINVAL }; 

    FILE* fp_ = NULL;
    int ec_ = fopen_s(&fp_, name_, mode_);

    // file is not opened for any of many reasons
    // the ERROR valstat make and return
    if (NULL == fp_)
        return { NULL ,  ec_ }; 
    
    // OK valstat state make and return
    return { fp_, 0 }; 
}
```
Very simple but safe, fully valstat enabled. The usage:
```cpp
// C++17 if() syntax
if (auto [ filep , errc ] 
    = modern_fopen( "non_existent_file" , "w+" ); 
   // return processing step one
   // check if value field is 'empty'
    filep )
{
   // step two:
   fprintf( filep, "non_existent_file opened" ) ;
}  
else {
   // return processing step one
   // has determined "value" field is empty
   // check the "status" field occupancy
   if ( errc )  {
   // step two begins here
   // status is not empty, we can use the content of the status field
   auto message = errc_to_string (errc);
   }
}
```
Above decouples from decades of "special return values" ,`errno` globals and POSIX "hash defines" lurking inside any C++ code base today. True that is done before. But, in addition to that, valstat enabled proxy functions, in front of the CRT legacy, are delivering resilient, uniform and interoperable solution, blessed by std lib too. And, available immediately. -->

## 8. Appendix: Requirements Common Across Domains

### 8.2. Interoperability

Each traditional solution to strict platform requirements is one nail in the coffin of interoperability. In danger of sounding like offering an panacea,  author will also draw the attention to the malleability of the valstat paradigm to be implemented with wide variety of languages used in developing components of an modern distributed system. 

Usability of an API is measured on all levels: from the code level, to the distributed system level. In order to design an API to be **feasibly** usable it has to be interoperable. That leads to three core requirements of

Interoperable API core requirements (to start with)

1. no "error code" as return value
   - Complexity arises from "special" error codes multiplied with different types multiplied with different context
   - In turn increasing the learning curve for every function in every API
   - How to decode the error code, becomes a prominent issue
     - Think Windows: `NTSTATUS`, `HRESULT`, `GetCode()`, `errno`
3. no special globals
   - Think errno legacy 
   - pure functions can not use globals

Some of the designed-in, simplicity in this paper is an result of deliberate attempt to increase the interoperability (also with other run-time environments and languages). 

It is important to understand there are inter domain interoperability requirements arising from inter language requirements. Examples: [WASM](https://developer.mozilla.org/en-US/docs/WebAssembly/C_to_wasm), [Node.JS](https://nodejs.org/api/addons.html), [Android](https://developer.android.com/studio/projects/add-native-code) and such. 

EOF