# Going the GO way

## Java Graal/Truffle

After writing a peg parser, some nodes and basics of interpreter I couldn't figure out why the Truffle DSL doesn't procude
the abstract node classes' factories as I should. I decided there is too much magic there for me. Too much of stuff that
works fast, but I don't exactly know how or can manipulate it. So I decided to go the GO route. Here the interpreter will be 
slower and making it fast will probably be much more code and work, but I will have hands on control to change anything, -- 
and learn much more.

## The Go version

I started with PEG parser. I found one, that works at runtime (not codegen), since we want to reuse it, to offer peg parsing
inside Rejy also then. I have defined a Rejy 0, which is a core of working interpreter with funciton calls and where I can test
the famous fibonacci function, to get basic orientation about where we are speed wise. Maybe manipulate runtime objects to 
improve it.
