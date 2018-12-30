## Learn Rejy in few steps

### Blocks, words, literal values

Rejy knows blocks

    { 1 2 "three" }
    { some words }
    { prind 10 }
    
Inside the blocks there can be words and literal values.

Rejy knows few different types of words:

    word // ordinary word can evaluate to value (or function - also value)
    setword: 2 + 5 // setword stores the result of expresion on the right of it 
    print setword // prints "7"

Program code consists of block filled with words and values. For example

    name: "Janko"
    print join "Hello " name
    // prints "Hello Janko"
    
Data in program is represented by blocks filled with words and values. For example:

{ name "Fifi" sound "Woof" age 5 }

or 

{ 'Tilly 'Sweat 'Hacksaw }

or

{ walk the dog in 10 minutes }

or 

{ load www.google.com and get title }

The last two ... are they data or code or both?
