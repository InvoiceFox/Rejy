## Trying out ideas

we focus on arrays of objects. Most big datasets imgo are arrays of relatively shallow objects. Many just one level. So 
this case should be elegant.

    { data: [ { name: "jim", age: 23 }, { name: "jane", age: 34 }, .... ] }
    
    ryk -j "{ 'data all 'name ( 'n print n ) }" 
    
    ryk -j "{ (sum: 0) 'data all 'age ( 'a sum: sum + a ) }"
    
    ryk -j "{ ( puts "Adults on J:" ) 'data where age > 25 .and starts-with "j" 'name ( 'n puts n ) }"
    
    
    { all_data: { people_data: [ { name: "jim", age: 23 }, { name: "jane", age: 34 }, .... ] }
    
    v003 ... litwords step into object properties, arrays have all, where <expr>, ... or we can use normal functions
    on blocks / series
    
    ryk -j "{ 'all_data 'people_data |first 'age |print } //
    
    ryk -j "{ 'all_data 'people_data |for-each 'x [ print x |get 'name ] }'
    
    ryk -j "{ 'all_data 'people_data |filter 'x get x 'age > 25 |forall 'x [ print x << 'name ] }'
    
Can lit'words be optional?
    
    ryk -j "{ 'all_data 'people_data |filter 'age > 25 |for-all [ print 'name ] }' ... litword also works as eval expression
                                                                                  ... this would be replicated in context of
Can we not use opwords but regular words that are used as opwords?    
    
    ryk -j "{ 'all_data 'people_data filter { 'age > 25 } for-all { print 'name } }" ... litword also works as eval expression
    
How do we get / process multiple values on same depth?

    ryk -j "{ 'all_data 'people_data for-all do { print jo_in 'name 'surname } }" ... litword also works as eval expression

Where example

    with person {
      join 'name 'surname
    }
  
    ; with takes block and insers the 'get object' in front of every lit-word
                                                                                  
with in this case would be loader word ... similar to factor's .. hm .. so we hove sort of macros .. the dialect above is just 
recursive with ... we aren't sure if we will have reader words yet.

## Another view at this

This is not just JSON dialec, this would also be general treestructure dialect for manipiulating Rye trees too.

We want multiple things all at once or separate with this dialect. 

 * navigation or collection / object tree
  * nicely determine particular paths inside a tree
 * extraction from collection / object tree
  * filter the collections, extract 
 * reshaping of objects
 
 Another take
 
 * navigating a array/object tree to particular fields / paths
 * filtering the collection based on values in objects (filtering)
 * reshaping individual objects (renaming, recalculating (joinint, ..), removing, passing thru, whitelist/blacklist)
 
 1) navigating
 
        /all_data/people_data/*/  -- all people
        /all_data/people_data/0/name -- name of first person
 
         vs.
        'all_data 'people_data all
        'all_data 'people_data first 'name
 
 2) filtering collections
 
        'all_data 'people_data 'name where starts-with "A"  -- should get just names starting with A
 
        vs
 
        'all_data 'people_data where 'name starts-with "A" -- should get people with names starting with A
         
         
        'all_data 'people_data where 'email exists and 'name starts-with "A"
        
 3) reshaping objects
 this is a combination of reshaping forms
 
        rename { 'name 'n 'surname 's }
        calc { full_name: 'name jo_in 'surname , birth: now/year - age }
        apply { name: uppercase }
        pass { 'name 'surname 'age }
        purge { 'email 'age }
    
 
        'all_data 'people_data where 'email exists and 'name starts-with "A" { pass { 'name 'age } apply { name: uppercase } }
        
        
So we have a workflow of navigation, filtering, reshaping. Looks quite sensible. If filtering could use normal functions it would be great, but maybe, specialised go-words would work faster. Once we are in a block we execute the reshape dialect which 
is a separate dialect for reshaping individual objects. The exact implementation of navigating and filtering also depends on the
type of JSON parser. Does it produce direct objects or does have callbacks in the middle of objects. Do we write our own?
        
## So how would this work (no exceptions yet)

 * lit-word goes into current object property at it becomes the current node
 * word is dereferenced
   * if it's a function it should call it
    * with current node as a first argument
    * if arguments to the right are lit words they get that key from current node
   * if it's a lit-word it's used as inline lit-word
   * if it's a lit-val it's unspecified for now (could replace current object or we say it's a stack and it's pushed on a stack)

This would get us:
    
    'all_data 'people_data 'name where-starts-with "A"
    
But how do we get to our goal (and later to combined expressions with and and or. Where is a function that takes a block and an code block (it's like filter).

    'all_data 'people_data all 'name where [ starts-with "A" ]

We were missing "all" there. So how would this work again ...

 * enters all_data on root object
 * enters people_data (array) on all_data object 
 * version A (all is not needed) --- now we have some sort of a loop because of array
   * all function takes array as fist argument and passes it through
   * because we are on array lit word takes each object and sets that as current node. So current node is now array of names names (strings)
   * where takes the array of strings and filters them by expression in block

This "Where" can't work in streaming / incremental way, because where takes whole array of objects and returns filtered array, so all must be loaded. If we want to just filter ... for example this methodology for 100000s object won't work as we want.

    ryk -j "{ 'name where [ start-with "A" ] }"

If we use the default go json parser in streaming way. We only process array of smaller object, objects are parsed in one swop, loaded to ram, only array is incrementally parsed. Case where there is one big object with one big array that we want to incrementaly parse are probably not that common. Still, the code should be uniform I think.

Basically this all depends now on how exactly do we parse streams of JSON. I think for start we should still limit to arrays
of objects, like awk is limited to lines of text.

Approach for reading continiously from stdin:
https://stackoverflow.com/questions/29421470/how-to-parse-an-infinite-json-array-from-stdin-in-go

Ok so our process is 

 * filter array (filter)
 * reshape objects (map)
 * aggrefate objects (reduce)
 
examples:

    { where 'name starts-with "A" }
    
on each object test expression, if it passes the condition print it

    { where 'name starts-with "A" { print uppercase 'name } }
    
on each object test expression, if it passes the condition then the block is called with current object the lit word in this
context automatically gets the property of the current object. What about not requiring a block and all words are infix?

    { where 'name starts-with "A" 'name uppercase print } }
    
where takes one opword expression "'name start-with "A"" after expression is exausted it either navigates or executes in infix
fashion. 'name sets string as current node. calls uppercase with that arg and sets new string as current node. print prints it.

lets try more complex expressions:

    { where 'children length? greater? 2 pass { 'name 'surname } }
    
 PROBLEM: but since all words are opword how does the program know where the expression ends ... it can't
 
 it could use a litword as a breaker, a comma, or a block.
 
    { where 'name starts-with "A" 'name uppercase print } }
    { where 'children length? greater? 2 'node pass { 'name 'surname } } // node is a predefined keyword - smell :/
    
    { where 'name starts-with "A" , 'name uppercase print } }
    { where 'children length? greater? 2 , pass { 'name 'surname } }     // looks ok, but we didn't yet even implement 
                                                                         // expression breaker "," so we can't know for 100% 
                                                                         // if it will be in program and as what
    
    { count: 0 } { where 'children length? greater? 2 , { count: inc count } nil } { count } // don't return anything per object, just count at the end
    
    
    { where 'name starts-with "A" { 'name uppercase print } } }
    { where 'children length? greater? 2 { pass { 'name 'surname } } } // overloading blocks

Expression terminator seems like the best option, lets walk with this further. What do we return /stream further.. it can be JSON. If value is returned it's streamed as node of JSON array further. If we explicitly print it, we print exactly that.

    { where 'children length? greater? 2 , pass { 'name 'surname } } // prints array with { name: surname: } objects
    
    { where 'children length? greater? 2 , 'name jo_in 'surname puts // prints "name surname"
    
nil prints nothing. block gets executed like regural code with lit-words still giving current object's properties
    
    { count: 0 } { where 'children length? greater? 2 , { count: count inc } nil } { count }

Do we have to make a special evaluator, or could we turn this into normal tree of code, that gets called on each object?
This is the next question. Or could this simply be special with dialect / execution model?

    with person {
      when 'name equals "Janko" ,
      'name print 'surname puts
    }

    with person {
      when 'name equals "Janko" and 'age > 40 ,
      inspect
    }
    
    with person {
      'name uppercase print
      'surname uppercase puts
    }

NEW THINGS: 
 * "when" is like if, but it just quits the execution of current block. This is a little weird, but it shows elegant 
 at least in cases above. It could be done as builting somehow probably.
 * "with" here provides all lit words with (get obj 'lit-word), which could probably done as a reader function .. but in this
 case we don't know if any of the words are not function but lit-words too. We will think about it later.
 * all words in this context are used as opwords ... is this bound to with, or could we use with with prefix words sometimes and 
 this would be usable in other cases too (most probably). What if we make the .{ } block modifyer, which this particular
 would at read time or runtime change the type of all function words in block.

WEIRD THOUGHT EXP. opwords have a conflict because they function only as functions but have no meaning as variables, but words that they denote can also be variables. Here we can't know at reader time if something refs a function or value or litword.

functions also have by our ideas different scoping rules than variables (need to be explicitly imported). 

So it would make by far the sense that functions would use different types of words than variables.... but but but...

    with person {
      when 'name equals "Janko" ,
      'name print 'surname puts
    }
    
    with person {
      when name equals "Janko" ,
      name print surname puts
    }
    
Hm ... it could be argued, that the first is more readable than the second :/

I found another problem. When and where upthere are not really opwords ... so not functions inthere are opwords which 
I am not yet sure how to solve with elegance and unifying approach.

It's cool, but do we need infix here really?

    with person {
      when equals? 'name "Janko" ,
      print 'name puts 'surname
    }
    
    { when starts-with 'name "A" , print uppercase 'name } }
    { when greater? length? 'children 2 , pass { 'name 'surname } }
    
This, especially last example is much worse ... we want infix ... do we move all to infix or infix by default?

    with person .{
      when| name equals "Janko" ,
      name print , surname puts
    }
    
Maybe we could have infix and prefix blocks and |word or .word forces infix and word. word| forces prefix.

IDEA: what if word that doesn't have values on the left, that is at the start of block or right after expression terminator.

## Solidification

I've implemented opwords, pipewords and expression guards yesterday. https://twitter.com/refaktor/status/1096083646019235840

Now that I look at examples above. We ditch .{ for now ... we also ditch word| for now. 

"with" function could change all regular words except first if there is one (up to expression guard) to pipewords and before all first litwords insert " 'name " " person |get 'name " . If there are consequetive litwords they should take previous object so it
just adds " |get 'first_name ".

So 

    { 'person 'name 'first_name uppercase } 
    // would become =>
    { tmpvar |get 'person |get 'name |get 'first_name |uppercase }

more complex scenario

    { 'person 'age > 30 and 'person 'name starts-with "a" .uppercase } 
    // would become => 
    { tmp |get 'person |get 'age > 30 |and tmp |get 'person |get 'name |start-with "a" .uppercase }
    
Not sure if all the leanings are correct here .. we should test it when possible. 

What prefix code would do the same?

    and > get get tmp 'person 'age 30  starts-with get get tmp 'person 'name uppercase "a"
    
This isn't good. Some parens would make it clear, but not really nice either:

    (and (> (get (get tmp 'person) 'age) 30)  (starts-with (get (get tmp 'person) 'name) (uppercase "a")))
    
Rebol has infix operators like > and all so it would be 2 cents better:

    all [ (get (get tmp 'person) 'age) > 30  starts-with (get (get tmp 'person) 'name) (uppercase "a") ]

## Next ...

I've made some quite big language design decisions this round. To soak this all in, see more examples, more implementation
before loading new design decision on top of it (like when (reader func ... or dual func? first time run (init) inited?) and tags, context words, function scope ... I should do something non designish. That would be adding support for strings and adding
some practical builtins for making ryk examples possible.

I could try to implement and test the json streaming reader inside ryk / stdio context, to see if it all works and maybe manipulate it a little. Maybe I get some new ideas along the way.

## Observation

Looking at our example, extended above ... the pipes in pipewords are a little too noisy. : would be better and symetric to ., but are already used for assignment, which is totally unrelated context and isn't compatible with this meaning.              
