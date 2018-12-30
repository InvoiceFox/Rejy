# Basics

I am writing down ideas about language design, as they come, here.

### What is Rejy

* First of all, it's a **experimental** or **toy** language made by amateur language designer
* A language that **looks** a little more familiar to general developers, but is 100% REBOL-like blocks (three legged stool poem)

### Trying to achieve
* rebol like blocks, words and values all the way down - NP
* advanced simple and safe objects with tags, prototypes (delegates) validators, transformators and multimethods - OK
* Multimethods on tags and natives - OK
* User definable opwords (infix words) - TRY

### Unresolved
* Scopes, isolation, pure-functions? read only outer access?? (can it be read only? if funcs aren't pure??)
* Pure funcs that cal only call pure funcs?
* Modules?
* Recursion? Trampoline?

### Hail to the block (of code or data)

    a: { this is "a block" 'of 6 :values }
    do { print 101 }
    repeat { print 'x }
    my-add: func { a b } { add a b }
    either equal? x 10 { "wow" } { "meh" }
     
Where *do*, *repeat*, *func*, *either* are all just normal functions you can make your own version of. *a:*, *my-add* are just 
specific types of words (setwords), that have specific behaviour in the **do** dialect.

## WIP Ideas

### Multimethods by first argument, tags, validations

Multimethods seem useful and help with using context-sensitive small words not AddWindonToApplication like long names. They nicely match with code outside objects/records/tuples and separation, late binding/manipulation between the two.

What do we dispatc on, since we don't have classes/types. We have adhoc objects/records ... we could set them TAG. There would be a (module/namespace/global environment) indexed registry of TAGs, and objects would hold the indexes. The multimethods would dispatch on those indexes.

Simple definition of object via function tuple.

    buyer: tuple { name: "Janko" age: 40 }

We can define tags, tags can have validation spec. 

    register-tag 'person { name: required and string age: optional 0 and integer birth: optional none and iso-date }

    tag buyer 'person
    
We can create object with a spec. Spec can include prototype object (instance - word) and/or tag (lit-word).

    customer: object { buyer 'person } { age: 30 }
    
    wife: object { 'person } { name: "Jana" birth: 1995-11-01 }
    
    wife2: object { wife } { age: wife|calc-age } // create new object with original wife as delegate prototype and age calculated by multimethod on 'person
    
    calc-age: multi { 'person person } { age: sub now|year person/birth|year } // | now|year => (year (now)) (pipe)
    
    calc-age: multi { 'person person } { age: sub now | year person/birth | year }
    
Here we have to decide, if we go for sorts of declarable (safer), but then const enviroments, or do we fully manipulate environments at runtime. Is the first the point or the second???
    
    +: multi { !number n m } { add n m }
    +: multi { !string s t } { join n m }
   
Multimethods should also work on opwords and native types.

### Infix notation - opwords

#### Why and how

We want the posible beautiful composibility and clarity of infix notation. We want infix notation to be the first class citizen.
There should be infix natives, native multimethods and infix user functions. Infix is nice and beautiful, but it has to be visually obvious to the reader of code without always knowing specific word in specific context. For this reason only opwords can be infix and opwors are or start with one of the typical infix operators (+ - * & > < ...)

#### Some examples

Math expressions are obvious:

    1 + 2          // 3
    5 * 10         // 50
    20 > 10        // true
    100 + 20 - 50  // 80

Opwords can be more than operators
    
    { 1 } +append 2            // { 1 2 }
    "janko" +join3 _ "metelko"  // "janko metelko"
    
Since they are multimethods they can also be ordinary operators:
    
    "janko" + " " + "metelko"   // "janko metelko"
    
What about piping of functions and postfix with one operator (see language like K and APL .... for examples) ... 
This is something we need to think - test - look 

    http://wwww.cebelca.biz |read |get-title |uppercase |print 
    
    
#### infix notation and opwords - WORKS                                                                                                                                                                                                                                  
                                                                                                                                                                                                                                                                       
opwords are words that start with a typical infix operator. like 1 + 2 ; 3 - 2 ; "janko" +join "metelko"                                                                                                                                                               
                                                                                                                                                                                                                                                                       
Are opwords explicit natives, it's own list or all diadic functions added with opword opeartor on front                                                                                                                                                                
                                                                                                                                                                                                                                                                       
1) opwords are defined as normal functions with opword format. They are always diadic                                                                                                                                                                                  
 + explicit clear nature                                                                                                                                                                                                                                               
 - lack of words, requrenment to define opword versions with same functionality as their prefix siblings                                                                                                                                                               
   ( THIS IS BAD IN FACT ... promotes duplication of code by default )                                                                                                                                                                                                 
                                                                                                                                                                                                                                                                       
2) any word that takes two arguments can be evaluated as opword if infix operator is added in front (+-&*=><~..)                                                                                                                                                       
 + no natives duplication, all diadic words are instantly usable = THIS ONE                                                                                                                                                                                            
                                                                                                                                                                                                                                                                       
~ is for neutral ... where + - etc don't make sense                                                                                                                                                                                                                    
                                                                                                                                                                                                                                                                       
    join "Jim" "Beam"  = = =  "Jim" +join "Beam"                                                                                                                                                                                                                       
                                                                                                                                                                                                                                                                       
    append block { 10 20 }  = = =  block +append { 10 20 }                                                                                                                                                                                                             
                                                                                                                                                                                                                                                                       
                                                                                                                                                                                                                                                                       
#### pipewords (monadic words - one argument ... postfix basically)                                                                                                                                                                                                     
                                                                                                                                                                                                                                                                       
pipewords are normal words that have | as a first character. The pipe is removed and a word without the pipe is found and used.                                                                                                                                        
                                                                                                                                                                                                                                                                       
    10 |inc |print                                                                                                                                                                                                                                                     
                                                                                                                                                                                                                                                                       
    http://www.cebelca.biz |read |parse-title |upper-case |print                                                                                                                                                                                                       
                                                                                                                                                                                                                                                                       
                                                                                                                                                                                                                                                                       
#### can opwords have more arguments than 2                                                                                                                                                                                                                             
                                                                                                                                                                                                                                                                       
Look at either in example below - it's an opword which takes condition on left and two blocks on right.                                                                                                                                                                
                                                                                                                                                                                                                                                                       
    block |length? ~greater? 10 ~either [ "greater ] [ "lesser" ] |print                                                                                                                                                                                               
                                                                                                                                                                                                                                                                       
**Reasoning**                                                                                                                                                                                                                                                          
                                                                                                                                                                                                                                                                       
pipewords and opwords are like shallow stack programming. but some words in a pipeline will need more than 1 or 2 arguments                                                                                                                                            
                                                                                                                                                                                                                                                                       
**How do we do it**                                                                                                                                                                                                                                                    
                                                                                                                                                                                                                                                                       
when processing opword we check how many arguments a function has. If it has 1 it's an error. If 2 it's normal opword, if more                                                                                                                                         
than 2 then after getting the right argument we loop further and collect more arguments on the right.  



 
    
### Pipe trick, infix opwords ... this would be nice (needs solving on evaluator side)
    
    
The pipe trick ... if we make evaluator with infixed operators and opwords as planned then this should be no special problem either.

    customer/name|to-uppercase | limit-length 10
    
    person/birth|year|to-string|wrap "***"|print

Definition of object with spec. Spec can have tag and validation, spec if bound to tag and can't be changed. Produces an error on collision.


## Implementation

### Representation of Rejy nodes in JS

Currently, each value in Rejy is small JS Array. First value is integer type, second is value, third and so on are optional.
```
{ word: 2 { janko } } // turns to

[ 1, // block
   [ 13, "word" ], // setword
   [ 101, 2 ],     // 
   [ 1,
      [ 11, "dog" ],
      [ 14, "woof" ]
   ]
]
```

### Internal representation of values

Problem: Some of functions we have now in Rejy work directly on unboxed JS values, for speed and simplicity purposes, but the AST Nodes aren't like that. 
* Solution would be that we would use direct JS values for Native types in AST. String, Number, Int .. what about arrays and objects? ... and are there any benefits of these two not being different syntactically in Rejy too? Object is not really series and functions on series don't really make sense on it or even work ... ? _ ? Minus is that then we don't have only series there
* Another solutio is that we transfer from AST to interpreter values and back (get AST, turn to array/object, manipulate in JS and return as AST Nodes again)
* Don't know what makes more sense ... maybe we try both... 
* It makes sense that the series as code is series, but  function turns it into object (they series functions make no sense any more - before that .. when it's still code/data series operations make full sense)

### Optimisation - native JS values instead of arrays like above
```
[ ] is block
"" is string
213 or 21.42 is number
new Uint16Array([11, <idx>]); word
new Uint16Array([12, <idx>]); setword
new Uint16Array([13, <idx>]); getword
new Uint16Array([14, <idx>]); litword

// functions tuple and object also generate rejy object which is JS object
{ }
```

```
{ word 2 { "dog" 'woof } } // is now 
[ // block
   [ { "0": 11 }, "word" ], // Uint8Array(11) is word
   2, // nuber
   [ // block
      "dog", // string
      [ { "0": 14 }, "woof" ] // Uint8Array(14) is litword
   ]
]
```
if we visit array we checj if first item is Uint8Array() if it is it's composite type. If it is we turn it into Uint16Array(2) where first value is type and second is index in process wide WORDSINDEX.  

    xx instanceof Uint16Array


### How do we go about this

Try to generate 

## Failed

### Objects and modules - FAILED

* Objects have a lookup list, list ob object prototypes where words are looked-up, if not found in current object.
* Modules have the same need of lookup, they usually lookup into modules they import. So here Modules are exactly just objects
  with same functionality

```
jim: tuple {
   name: "Jim"
   age: 30
}
```
*tuple* is a function that creates object without spec, *object* is a function that creates object with spec also. Spec has it's 
own spec dialect. If it's just list of words it means these are lookup objects. With a little help from setwords you can add
additional spec types.

    livething: tuple {
        born: none
    }

    mammal: tuple {
        sound: none
        say: does { print join "dog says " sound }
    }
    
    dog: object { mammal livething } {
        sound: "woof"
    }

**FAIL ... we don't want to use "object/method args" code at all ... basically we just want data tuples and multimethods**

