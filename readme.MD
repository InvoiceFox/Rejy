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

