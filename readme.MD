# Basics

I am writing down ideas about language design, as they come, here.

### What is Rejy

* First of all, it's a **experimental** or **toy** language made by amateur language designer
* A language that **looks** a little more familiar to general developers, but is 100% REBOL-like blocks (three legged stool poem)
* Simple prototype inspired object model (no classes - for now no methods)
* Multimethods - tags - validations 
* Hopefully stronger functional leaning
* Homoiconic (code and data are words, values and blocks all the way ... no keywords, no special forms)

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
    
    
The pipe trick ... if we make evaluator with infixed operators and opwords as planned then this should be no special problem either.

    customer/name|to-uppercase|limit-length 10
    
    person/birth | year | to-string | wrap "***"



Definition of object with spec. Spec can have tag and validation, spec if bound to tag and can't be changed. Produces an error on collision.

    buyer: object { 'person name: required and string age: optional 0 and integer }
              { name: "Janko" }



### Objects and modules

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
