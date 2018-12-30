## Rejy basics


    *this document tries to explain what and why to people not familiar with Re(db)ol languages*


Rejy is an experimental language. I took the basic idea of Rebol, made it visually closer for most programmers and am extending it in some new ways to see where it leads. 

Current implementation is an interpreter in JavaScript. Dart or JVM interpreter is in plans IF it shows worthy. Contrary to Re(db)ols, the practical focus will be on backend / console programming / IO, not the GUI-s.

Here you can see some similar (more developed) languages of this ilk:
  *  www.rebol.com (Rebol 2 - closed source language that started it all)
  *  www.red-lang.com
  *  https://github.com/metaeducation/ren-c 

Rejy is a dynamic, fully homoiconic `toy` language. Lisp met forth in a dark alley and lost the parenthesis and the stack shuffle words.

### Blocks, words and values

Everything in Rejy resides in blocks:

    { 1 2 "three" }
    { some words }
    { print 10 }
    
Inside the **blocks**, there can be **words** and **values**.

Rejy knows few different types of words. Let's met the first few:

    a-word      // an ordinary word
    setword:    // a set-word
    'lit-word   // literal word
    +           // opword
    
Rejy knows many types of literal values:

    123 2.512               // numbers
    "some text"             // string
    http://www.cebelca.biz  // url TODO
    support@example.com     // email TODO
    %/path/to/file          // path TODO
    ...
    
These are few examples of Rejy blocks:

    { 'Tilly 'Sweat 'Hacksaw }
    { name: "Fifi" sound: "Woof" age: 5 }
    { url http://usrjoy.com last-check 2018-12-30T20:21:00 }
    { ping me about "appointment with Jim" in 10 minutes }
    { page: load http://www.google.com print get-title page }
    { root { branch1 {} branch2 { sub-branch1 } } }

If you look - some of these look like data some like code and both claims are correct. 

Only the dialect, when it evaluates the block determines how certain word and value types act.  

### The DO dialect

DO dialect takes block of words and values and evaluates - or **does** it. DO dialect is the default programming language/dialect in Rejy. Words like **join**, **print** are functions that accept fixed number of arguments. **do** itself is a function of do dialect.

    do { "Tilly" }                      // returns "Tilly"
    do { join "Danne" "mora" }          // returns "Dannemora"
    do { print "Hello world!" }         // prints "Hello world!" returns none

Words hold references to values. Functions are also values. Set-word takes the value on it's left and assigns it to the word.

    name: "Sunny"               // set-word name: assigns expression on the left to word *name*
    print name                  // prints "Sunny" - print is a word holding a function that prints value it's passed
    
    surname: join "The " "Kid"  // join is a function taking two string arguments and joining them , setword surname: assigns 
                                // the value to word surname
    print surname               // prints "The Kid"

Blocks with the DO dialect code are no different than blocks with other data. Rejy has many functions that help you work with blocks.
 
    append { "apple" "orange" } "lemon"                 // returns { "apple" "orange" "lemon" }
    second { "apple" "orange" }                         // returns "orange"
    do append { print } second { "apple" "lemon" }      // prints "lemon"

DO dialect has no keywords or special forms. Things like **if**, **loop**, **for** are just ordinary functions.

    if greater? 10 5 { print "Woof!" } // prints "Woof!"
    
    loop 3 { print "*" } 
    // prints
    *
    *
    *
    
    for person in { 'Tilly 'Sweat 'Hacksaw } { print join "Why " person }
    // prints
    Why Tilly
    Why Sweat
    Why Hacksaw
   
**if** for example, is just a function that accepts two arguments: a boolean and a block of code to execute if the first argument is true. For is similar:

    just-a-block: { print "Woof!" }     // we assign the block to a word just-a-block
    if greater? 10 5 just-a-block       // prints "Woof!"
    
    for person in { 'Tilly 'Sweat 'Hacksaw } append { print join } { "Why " person }
    // prints
    Why Tilly
    Why Sweat
    Why Hacksaw
    



 ***... work in progress ...***
    
