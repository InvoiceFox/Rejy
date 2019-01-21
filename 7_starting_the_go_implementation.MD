# Going the GO way

## Java Graal/Truffle

After writing a peg parser, some nodes and basics of interpreter I couldn't figure out why the Truffle DSL @GenerateFactoryClasses doesn't produce the abstract node classes' factories as I should. I decided there is just too much magic there for me. Too
much of stuff that makes it work super fast, but I don't exactly know how or can manipulate it. So I decided to go the manual GO route. Here the interpreter will be slower and making it fast will probably be much more code and work, but I will have hands on control to change anything, -- and learn much more.

## The Go version

I started with PEG parser. I found one, that works at runtime (not codegen), since we want to reuse it, to offer peg parsing
inside Rejy also then. I have defined a Rejy 0, which is a core of working interpreter with funciton calls and where I can test
the famous fibonacci function, to get basic orientation about where we are speed wise. Maybe manipulate runtime objects to 
improve it.

## Go benefits and small reorientation - Rejywk

Since Go produces native bindings without the weight of JVM this interpreter could be good for command line use. I love command
line tools, like awk, but I want something better, maybe more structured, that can turn stream of text to stream of json for example and then manipulate that. Anyway, I believe everything a person does must have a concrete usable target to strive to, and 
to me right now it to make an **awk I always wanted**. This will be possible much earlier than anything else, let alone long 
running server apps, which is the ultimate practical goal. 

Rejywk - rjk in short will try to use infix postfix words, dialecting, etc ... 

    # infix postfix words to the rescue or at least to some better left-to-right maleability
    cat data | rjk -s, -b'{ $1 |greater? 1990 |either [ $2 ] [ $3 ] + 100 |print }'

    # awk for json data -json loads one object 
    cat data.json | rjk -json '{ now |year - ($$ >get 'person >get 'birth |parse-date |year) ^+_ ($$ >get 'person >get 'name) |print }'
    # dialect or just ordinary functions to turn csv to json, you can also turn and then to manipulationg -json further
    cat data | rjk -tojson -s; '{ make { type: 'person birth: $2 name: $4 } }'
    
Rjk has multiple profiles with modules and helper functions that you can edit and prepare and can load via flag.
