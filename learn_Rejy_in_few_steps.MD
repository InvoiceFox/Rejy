## Learn Rejy in few steps

### Blocks, words, literal values

Rejy understands blocks

    { 1 2 "three" }
    { some words }
    { prind 10 }
    
Inside the blocks there can be words and literal values.

Rejy knows few different types of words. Let's met the first two:

    a-word // an ordinary word
    setword: // a set-word
    'lit-word  // literal word
    
Rejy knows many types of literal values:

    123 2.512 // numbers
    "some text" // strings
    http://www.cebelca.biz // urls
    ...
    
So let's see few example blocks:

    { 'Tilly 'Sweat 'Hacksaw }
    { name: "Fifi" sound: "Woof" age: 5 }
    { walk the dog in 10 minutes }
    { load http://www.google.com and get 'title }
    { root { branch1 {} branch2 { sub-branch1 } } }

### The DO dialect

Rejy blocks can hold any mix of words and values, but some subset makes sense to the DO dialect. Do dialect is what could be called the default programming language of Rejy, but there can be others. 

    do { "Tilly" } // return "Tilly"

    do { join "Danne" "mora" } // returns "Dannemora"
    
    do { print "Hello world!" } // prints "Hello world!"
    
Blocks with the DO dialect code are no different to the Rejy than blocks with other data. Rejy has functions that help you work with blocks.
 
    append { "apple" "orange" } "lemon" // returns { "apple" "orange" "lemon" }
    
    second { "apple" "orange" } // returns "orange"
 
    do append { print } "lemon" // prints "lemon"

The DO dialect has no keywords or special forms. Just words that evaluate to functions. These are usually keywords or special forms.

    if greater? 10 5 { print "Woof!" } // prints "Woof!"
    
    loop 3 { print "*" } 
    // prints
    *
    *
    *
    
    for person in { 'Tilly 'Sweat 'Hasksaw } { print join "Why " person }
    // prints
    Why Tilly
    Why Sweat
    Why Hasksaw
   
But if, loop and for aren't keywords. They are just functions, that you can make on your own. If for example, is just a function that accepts two arguments: a boolean (true / false) and a block of code to execute if the first argument is true. There is no special form for all this either. Look at these:

    just-a-block: append { print "Woof!" } // we assign the block to a word just-a-block
    if greater? 10 5 just-a-block // prints "Woof!"
    
    for person in { 'Tilly 'Sweat 'Hasksaw } append { print join } { "Why " person }
    // prints
    Why Tilly
    Why Sweat
    Why Hasksaw
    
### The DO dialect and set-words

Set-words are just specific type of words. We call them like this, because in DO dialect they set values to the words.

    a: 10
    b: 5
    c: sum a b
    print c // prints 15
   
   
