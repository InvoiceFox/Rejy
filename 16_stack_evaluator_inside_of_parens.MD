# Got a little weird idea

I got this idea the other day. if we would like to mimic classical algol like languages in a weird way we could do this. The idea itself is maybe just dubm, but I was always tempted in introducing stack like language as a part of Rejy. So maybe this is some entry idea ...
   
    join ( a b )
    
    // would work like this: 
    
    // join would see it's first argument is an parenthesis so it would run the code inside in stack mode
    // only, and from stack fill function's args. Idea was that functions could operate on stack instad on
    // full context either way. So simple functions would be compiled from named access to stack based code
    // which could be more lightweight. No creating and destroying of context on each function entry.
    
This would make anything inside () run in stack mode. 

NEGATIVES:

this conflicts with math notation, grouping. And conflicts if we want to group other functions in custom lisp like way.

    (add (inc a) (subtract 5 2)) We loose this :/ ... 
    
But let's play along either way. (we can change this later)

So what would this bring us ... if anything.

    ( a b add )

    ( a b add 2 add )

    ( a b add 2 5 add a: b: )

Stack could come handy if functions returned multiple results on a stack (?) or in parse PEG / collect / maybe .... 

Java PEG dialect used stack to collect values together. 

Stack evaluator could be more optimal at more LW function calls ... half way to vm-like stack based evaluator (??)

Factor-lang compiled use of lexical words to stack operations. We could try similar.

... This page is dedicated for ideas about stack mode ...

## Multiple return values

    [( get-person-info age: name: )]
    
Maybe not the most elegant ... what if all functions returned to stack ... so we could do jump into stack mode at any time.

    get-person-info ( age: name: )
   
    get-person-info [( age: name: )]
    
    get-person-info '( age: name: )
    
    get-person-info do-stack ( age: name: )
    
    
   
