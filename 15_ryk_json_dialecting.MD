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

      '{ where 'name starts-with "A" }'
