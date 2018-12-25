# Basics

I am writing down ideas about language design, as they come, here.

### What is Rejy

* First of all, it's a **experimental** or **toy** language made by amateur language designer
* A language that **looks** a little more familiar to general developers, but is 100% REBOL-like blocks (three legged stool poem)
* Hopefully simple prototype inspired object model (no classes)
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

### Objects and modules

* Objects have a lookup list, list ob object prototypes where words are looked-up, if not found in current object.
* Modules have the same need of lookup, they usually lookup into modules they import. So here Modules are exactly just objects
  with same functionality


```jim: tuple {
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
