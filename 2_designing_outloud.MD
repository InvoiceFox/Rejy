# Writing down basic ideas of language

I am writing down ideas about language design and implementation, as they come, here.

### What is Rejy

* First of all, it's a **experimental** or **toy** language made by **amateur** language designer
* A language that **looks** a little more familiar to general developers, but is 100% REBOL-like blocks 
    (three legged stool poem)
* If and where possible, friendlier to FP concepts, safer

### What we want

* rebol like blocks, words and values all the way down - DONE
* Multimethods on tags and natives - DONE
* User definable opwords (infix words) - DONE
* CLOS like prototypical object system 

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

    customer/name |to-uppercase |limit-length 10
    
    person/birth |year |to-string |wrap "***" |print

Definition of object with spec. Spec can have tag and validation, spec if bound to tag and can't be changed. Produces an error on collision.  

### Lit-words

Lit-words evaluate to word
    
    do [ 'word ] // word
    
    print 'word  // prints word
    

### Tags-words

We need tag-words to have special behaviour. They evaluate to themselves, so the asignment holds value/idex of assigned word, not it's valie in way.  To really evaluate them we need get-tag-word.

    <tagtag> // no undefined error, evaluates to itself (indexes the tagtag)
    <tagtag> tag { ... spec ... }
    // so when we call
    jim: object <tagtag> { }    // <tagtag> doesn't evaluate to value, but to indexed word tagtag itself 
                                //which we need in this case.
    
    <tagtag>: 500
    x: 333 + :<tagtag> // only get-tag-word returns value 888
    
Could also get helpfull for constructs like 

    wrap { <h2> "Ljubljana" <p> "some text" }   // where <strong> and <p> are undefined, but return themself and are indexed

### tuple, object

Currently we want this:

    // tuple function creates object without tag
    car: tuple { brand: "Audi" }

    // we create a tag .. simple spec dialect will work in version 0.02
    <person>: tag { }

    // object creates object with tag>
    jim: object <person> { name: "Jim" age: 40 }

Then we can make a generic user function on a tag

    // we need to make lit-words work and a method that get's object property based on lit-words
    jim .get 'name  // returns "Jim"

    introduce: generic { <person> p } { p .get 'name |print }
    
Setword assignment will not work in this case, because we don't set value of multimethod directly, but we register it with generic method object or with some per module global list, which won't work well as it can't be naturally scoped etc. So it must be some local object, similar to CLOS and Factor. 

When we call the function, we don't call it, but we call this local generic object and it then dispatches based on first argument. I am not sure... what happens when jou include module with multimethods objects that you already have? If you do it into same namespace ... do they get automatically merged into your local multimethod object? :/ What if it's not in top-level code? ... 

    module1
      intro: generic {}
      beep: generic {}
      method 'intro { <person> p } { print get p 'name }
      method 'intro { <car> c } { print get c 'brand }
      method 'beep { <car> c } { print "beep!" }

    module2
      intro: generic {}
      move: generic {}
      method 'intro { <house> h } { print get h 'address }
      method 'move { <car> c } { print h .get 'miles .set-to h 'miles }
    
The point of generic methods is that we do call short not namespaced words that dispatch on first arg ... so ehat is most natural here??

look at clos, dylan, factor for modules+scopes+generic methods.

