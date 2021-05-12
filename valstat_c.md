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
  - [3.1. Type requirements](#31-type-requirements)
  - [3.2. bare-bones valstat](#32-bare-bones-valstat)
- [4. Conclusions](#4-conclusions)
- [5. References](#5-references)
- [6. Appendix A](#6-appendix-a)
  - [6.1. Fully functional but lightweight valstat type](#61-fully-functional-but-lightweight-valstat-type)
<h2>&nbsp;</h2>

## Revision history<!-- omit in toc -->

R0: Created

<!-- div class="page"/ -->

## 1. Abstract

This is a proposal about logical, feasible, lightweight and effective handling of information returned from C functions, based on the [valstat protocol](https://github.com/DBJDBJ/valstat).

valstat is not yet another error handling idiom. Please take a slight and quick detour to read that document first.

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

### 3.1. Type requirements

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

### 3.2. bare-bones valstat 

It is quite ok and enough to be using fundamental types for both value and status fields. We can implement the "field" paradigm by using just that: C fundamental types. Example

```cpp
   // 
   typedef struct {
      int value;
      int status;
   } valstat_int_int ;
```
Both value and status fields in here are just integers. Yet we can simply and comfortably represent four valstat states.
```cpp   
// create the state to be returned
valstat_int_int vii_ok    = { 42, 0 }; // OK state
valstat_int_int vii_error = { 0,  error_code }; // ERROR state
valstat_int_int vii_info  = { 0xFF, status_code }; // INFO state
valstat_int_int vii_empty = { 0, 0 }; // EMPTY state
```
The requirement of "emptiness" work perfectly well on the above valstat.
```cpp
valstat_int_int valstat = some_valstat_enabled_api();
int value = valstat.value;
int status = valstat.status;

// decode the state returned
// both occupied
  if (   value &&  status ) { /* state: info */ }
// occupied and empty  
  if (   value && !status )  { /* state: ok   */ }
// empty and occupied  
  if ( !value  && status )  { /* state: error*/ }
// empty and empty  
  if ( !value && !status )  { /* state: empty*/ }
```
It all depends on the context. That is still valstat protocol in action. Thus in some situations valstat field types can be two simple integers. 

Above is rather important valstat ability to be transparently adopted for various projects. That solution is not using any special types and can work under extremely strict runtime conditions. 

In some different context different valstat struct might be used.
```cpp
   // 
   typedef struct {
      some_struct * value;
      complex_status * status;
   } complex_valstat ;
```   
Still users will be using that as ever before, to make and decode the state returned.

<!-- div class="page"/ -->

## 4. Conclusions

*Fundamentally, the burden of proof is on the proposers.* — B. Stroustrup

"valstat" protocol is multilingual in nature. Thus adopters from any imperative language are free to implement it in any way they wish too. The key protocol benefit is: interoperability. 

Using the same protocol implementation it is feasible to develop standard C++ code using standard library, but in restricted environments. Author is certain readership knows quite well why is that situation considered unresolved in the domain of ISO C++. 

Authors primary aim is to propagate widespread adoption of this paradigm. As shown valstat protocol implemented is more than just solving the language agnostic "error-signalling problem". It is an paradigm shift, instrumental in solving the often hard and orthogonal set of platform requirements.

Of course, valstat protocol and C definition, while imposing extremely little on adopters is leaving the non-adopters to "proceed as before".

Obstacles to paradigm adoption are far from just technical. But here is at least an immediately usable attempt to chart the way out.

----------------------

<!-- <div class="page"/> -->

## 5. References

<a id="REF_STDC">[STDC]</a> 
ISO/IEC 9899:2018 -- Information technology — Programming languages — C, https://www.iso.org/standard/74528.html

- <a id="ref_modernc">[MODERNC]</a> Modern C, Jens Gustedt, https://modernc.gforge.inria.fr

- <a id="ref_empty">[EMPTY]</a> "Your Dictionary" **Definition of empty**,  https://www.yourdictionary.com/empty


<!-- div class="page"/ -->

## 6. Appendix A

*To me, one of the hallmarks of good programming is that the code looks so simple that you are tempted to dismiss the skill of the author. Writing good clean understandable code is hard work whatever language you are using -- Francis Glassborow*

Value of the programming paradigm is best understood by seeing the code using it. The more the merrier. Here are a few more simple examples illustrating the valstat protocol and implementations applicability.


### 6.1. Fully functional but lightweight valstat type

Let us assume we need to write a layer of safe proxies to some C run time (CRT) functions. We might use various valstat structs where the status field is actually the `errno` value. We can declare them by hand or possibly using simple macro

```cpp
// 
typedef int errno_field_type ;

// were status is always the same type
// aka errno_field_type
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
```c
typedef struct {
	int   value;
	errno_field_type  status;
} int_errno_valstat ;

```
But for the sake of clarity we shall avoid macros.
```cpp
typedef struct {
	int   value;
	errno_field_type  status;
} int_valstat ;
// 
typedef struct {
	int   value;
	errno_field_type  status;
} char_ptr_valstat ;
//
typedef struct {
	double *   value;
	errno_field_type  status;
} double_ptr_valstat ;
```
Those valstat variant might be used in a myriad of API's, internally facing the CRT, or CRT like API's. All returning the information in the shape of state + value, carried by the very few valstat declarations. Examples:

```c
char_ptr_valstat safe_read_line ( FILE * )  ;

double_ptr_valstat safe_sqrt( double * ) ;

char_ptr_valstat safe_strdup( const char * )  ;
```
The caller using any of the above imagined crt proxy functions, will follow the same valstat two step state capturing idiom, for returns processing. 

Next, let's see how modern_fopen(), might be actually implemented.

```c
//
typedef struct {
	FILE *   value;
	errno_field_type  status;
} f_ptr_valstat ;
// 
inline f_ptr_valstat 
  modern_fopen(const char* name_, const char* mode_) 
{
    // the ERROR valstat is returned on bad arguments
    if (NULL == name_)
        return (f_ptr_valstat){ NULL ,  EINVAL }; 

    if (NULL == mode_)
        return (f_ptr_valstat){ NULL ,  EINVAL }; 

    FILE* fp_ = NULL;
    int ec_ = fopen_s(&fp_, name_, mode_);

    // file is not opened for any of many reasons
    // the ERROR valstat make and return
    if (NULL == fp_)
        return (f_ptr_valstat){ NULL ,  ec_ }; 
    
    // OK valstat state make and return
    return (f_ptr_valstat){ fp_, 0 }; 
}
```
Very simple but safe, fully valstat enabled usage:
```cpp
f_ptr_errno_valstat fpev = modern_fopen( "non_existent_file" , "w+" ); 
FILE * value = fpev.value ;
errno_field_type status = fpev.status

// decode the state returned

  if (   value &&  status ) { /* state: info */ log("Miracle!") ;  }
// occupied and empty  
  if (   value && !status )  { /* state: ok   */ log("Miracle!") ; }
// empty and occupied  
  if ( !value  && status )  { /* state: error*/ log( errc_to_string (status))) ; }
// empty and empty  
  if ( !value && !status )  { /* state: empty*/ log("Expected") ; }
```
Above decouples from decades of "special return values" ,`errno` globals and POSIX "hash defines" lurking inside any C/C++ code base today. True similar is done before. But, in addition to that, valstat enabled proxy functions, in front of the CRT legacy, are delivering resilient, uniform and interoperable solution.
<!--
### 6.2. Interoperability  Across Domains

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

It is important to understand there are also interoperability requirements arising from different domains all residing on the same platform in the same time. 

Example: [WASM](https://developer.mozilla.org/en-US/docs/WebAssembly/C_to_wasm), [Node.JS](https://nodejs.org/api/addons.html) and [.NET Core](https://en.wikipedia.org/wiki/.NET_Core). All running in the same time on the same client machine.
-->
EOF
