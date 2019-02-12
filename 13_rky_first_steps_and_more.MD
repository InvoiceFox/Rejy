## Ryk - very first version

I made first version of ryk work. Rejy has only integers now, and this is really minimal, but it can grow from here now.

Example:

    $ cat > ints.txt
    1
    2
    3
    4
    5
    $ cat ints.txt | ./ryk "{ puts add inc 100 line }"
    102
    103
    104
    105
    106
   
When I update the parser, so words won't only consist of ascii chars I will add numbered arguments. Instead of $1 which look
noisy to me I am thinking about _1 _2 _ (for all)

I will also add the steps start, (main), end and separator definition then.

I am thinking if in-context words will help with tearsnes of code ... and how we woul for example generate json via awk:

    cat ints.txt | ./ryk "start { j: new-json-array } { add j line } end { j }"

This is nothing special. Let's try to parse csv (name, surname, age) to json objects { name: surname: birth: }

    cat people.csv | ./ryk "
    start { j: json-array } 
    { o: json-obj , set o 'name _1 , set o 'surname _2 , set o 'birth now/year - age , add j o } 
    end { j }"
    
The simple manual way above. The , as a expression guard looks really usefull and nice. Otherwise expressions above would be very hard to comprehend and two spaces don't help much.

    cat people.csv | ./ryk "
    start { j: json-array } 
    { add j json-obj-from csv-line _ '; [ 'name 'surname <birth> { now/year - _3 } ] } 
    end { j }"

more structured and declarative, but also more complex. The main problem I see in this code is hidden nesting which makes it quite incomprehensable.

## Making it nicer

let's add grouping for clarity _visible but verbose_

    cat people.csv | ./ryk "
    start { j: json-array } 
    { add j ( json-obj-from ( csv-line _ ', ) [ 'name 'surname <birth> { now/year - _3 } ] ) } 
    end { j }"

opwords probably wouldn't help much with clarity here... and because of left to right execution this wouldn't work. _just weird_

    cat people.csv | ./ryk "
    start { j: json-array } 
    { j *add ( _ *csv-line ', ) *json-obj-from [ 'name 'surname <birth> { now/year - _3 } ] } 
    end { j }"

Does turning of arguments help us any? Let's say that ** switches the first two arguments around _no_

    cat people.csv | ./ryk "
      start { j: json-array } 
      { (_ *csv-line ',) *json-obj-from [ 'name 'surname <birth> { now/year - _3 } ] **add j } 
      end { j }"
      
Would stack help? _no_
      
    cat people.csv | ./ryk "
     start { j: json-array } 
     { csv-line _ '; |push , json-obj-from pop [ 'name 'surname <birth> { now/year - _3 } |push , j +add pop  ] } 
     end { j }"

Temporary variables? _better than stack_
     
     cat people.csv | ./ryk "
      start { j: json-array } 
      { l: csv-line _ '; , o: json-obj-from l [ 'name 'surname <birth> { now/year - _3 } , j +add o  ] } 
      end { j }"
     
Currying / compose ?

     cat people.csv | ./ryk "
      start { 
         j: json-array 
         process: compose { csv-line '; ? , json-obj-from [ 'name 'surname <birth> { now/year - _3 } ? , add j ? ] }
      } 
      { process _ } 
      end { j }"
      
Hm ... this last one is some form of opwords .. where active argument is first. Let's say that _ is set as arg on the left already ... this is same form but looses the ?, compose and temp process:

     cat people.csv | ./ryk "
      start { j: json-array } 
      { |csv-line ';
        |json-obj-from [ 'name 'surname <birth> { now/year - _3 } ]
        |add* j ] } } 
      end { j }"

add here only makes sense in taking the json-array first and object second. Also so because it dispathes based on first arg. 
To fully utilize this kind of coding we would need arg manipulators ... right now * seems ok-ish. One star means that second arg
on is moved to first position ... ** would mean that third is moved, and rest are pushed forward. This must work with ordinary
words and opwords. It must of course still dispatch on the initially fist argument!

    sub 100 10 |asserts 90
    sub* 10 100 |asserts 90
    100 -sub 10 |asserts 90
    10 -sub* 100 |asserts 90
    
    join3 "ja" "nk" "o" |asserts "janko"
    join3* "nk" "ja" "o" |asserts "janko"
    join3** "nk" "o" "ja" |asserts "janko"
    
so opwords wirth arg switchers can be usefull.

## Manipulating functions

### removing named context in functions

I don't see a particular problem, why we couldn't at runtime change function so that it uses a stack instead of context _map_ or a simple small array for local words/variables. These both should be more optimal. With regards to stack we could just have one stack, pop values on and clean them if needed when function exits. Or we could leave things (or just result) on if it makes sense? For functions with small numer of local variables this should be faster than map.

### composing functions at source level

I also don't see why we couldn't take two functions and compose them together into a new function. But not just where they represent one flow of arguments (via curry) but into a new function that can take multiple missing arguments from composed ones.

    send-email: fn [ addr subj cont ] { if-addr addr [ send-eml addr join subj cont } } 
    get-user-email: fn [ id ] { db@proc |sql "select addr from user where id = ?" { id } |first-opt }
    
    send-email-to-user: fn { id subj cont } { addr: get-user-email id , send-email addr subj cont }
    // 
    send-email-to-user: compose-fn { get-user-email <id> , send-email _ <subj> <cont> }
    
    send-email-to-user: compose-fn { get-user-email ? , send-email _ ? ? }
    
    // we get a new function at source level (not linked to those two)
    source? send-email-to-user
    send-email-to-user [ id subj cont ] {
        f0001: do { db@proc |sql "select addr from user where id = ?" { id } |first-opt }
        if-addr f0001 [ send-eml f0001 join subj cont  
    }

Not sure, but would this be solved via currying by closures and composing the function at runtime when subj and cont are 
avaliable in local lontext? 

### partial application (currying) at source level, as part of evaluator

    send-email: fn [ addr subj cont ] { if-addr addr [ send-eml addr join subj cont } } 

    send-email-alert: send-email ? "alert" ?

### partial application as a word

    send-email-alert: partial { send-email ? "alert" ? }
    
    // vs
    
    send-email-alert: fn { addr cont } { send-email addr "alert" cont }
    
    // produces if in source level
    
    fn { addr cont } { subj: "alert" ... }
    
    // or a partial application object, that has a list of args and a link to function in question
    // when evaluated this object evaluates just args not yet provided and calls a function normally
    // maybe the same object could be used for composition?
    // if would hold a list of functions and already provided arguments and some way to know to which
    // argument the result of previous function applies
    // [ { args: { } fn: &get-user-email }, { apply-to: 0 fn: &send_email } ]
    // [ { args: { 1: "alert" } fn: &send-email } ]
    
    // would this be faster? ... maybe we could compile whole code series to these objects as an optimisation?
    
    // otherwise composing functions on a source level would also make them faster and maybe offer some ideas for 
    // optimisation. It's like inlining ... we could detect composable sequences on hot paths and then source 
    // compose them.
    
    // IDK ? :) ... it would be interesting to make a proof of concept for these ...                

## JSON processing

There are some go json (streaming) libraries, but no cake. The closes comes this one, which could be taken as base to 
build upon: https://github.com/buger/jsonparser

or the default one: 
https://stackoverflow.com/questions/31794355/stream-large-json

    func main() {
        var val interface{}

        decoder := json.NewDecoder(bytes.NewBuffer([]byte(sourcedoc)))
        decoder.Token() // consumes the first "["

        decoder.Decode(&val)
        fmt.Printf("First: %#v\n", val)
        decoder.Decode(&val)
        fmt.Printf("Second: %#v\n", val)
        for k := range val.(map[string]interface{}) {
	        fmt.Println(k)
	    }
        fmt.Println(val.(map[string]interface{})["colors"].([]interface{})[0])
        decoder.Token() // consumes the last "]"
    }

Some examples of streaming api-s dsl-s: 
https://www.npmjs.com/package/JSONStream ... event/sax like somewhat
http://oboejs.com/examples#transforming-json-while-it-is-streaming ... string based dialect
http://hackage.haskell.org/package/json-stream ... haskel, event internal dialect like
https://stedolan.github.io/jq/tutorial/

### Ideas

    ryk -j '{ in 'data all 'people { 'p puts get p 'name } }' // print all names in { data: { people: [ { name: "jim" } .. }
    
    ryk -j '{ in 'data all 'people { 'p puts join_ get p 'name get p 'surname } }'
    
    ryk -j '{ ( cnt: 0 ) in 'data all 'people get 'name ( 'n make-obj { person: n, id: cnt cnt: inc cnt } )' ...
    
    try further later ...
    
    
 
