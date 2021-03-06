# Tuples (objects / classes like things)

## So what is the main idea.

* We want to have composed objects (from multiple other objects)
* We want that a validation dialect can be applied to them

**kindered tuples**

* we want these objects to be set a kind
* we want to define a validation rules for each kind
* we want to be able to define conversions for kinds (from = to)

**everything has kind**

* also native singular object types should have a predefined kind (number, block ...)
* validation rules define the kind of tuple part and can additionally define requred/optional X , custom filters (one-word, email, ...)
  and custom calculation () and custom check ()

**we dispatch on kind of first argument**
* all functions should dispatch on kind of first arg so all native words can be used in other contexts (kinds), like add, first, +,  etc
* this means that there is one global registry of functions with kinds

### example of unkinded / adhoc validation

    load-data user-id #> { email: required email username: required one-word } .either { .save %user.ryet } { .print }
    
#### what is happening here
    #> is opword ( <obj> _#> <validation rules> , return validated object or validation-error object )
    
    { ... } are validation rules
    
    .either are basically two functions one is bound to kind org.ryelang/validated-tuple another is to org.ryelang/validation-errors
    first does first block, second just the second block. They set it's first argument as an entry (left) value. First turns it to 
    org.ryelang/tuple
    
    .save is bound to org.ryelang/tuple and molds and saves the tuple
    .print is bound to org.ryelang/validation-errors and formats and prints them
    
    #> with first arg kind block would take just block
    #> with first arg kind #tag would take tag and block. If validation passes 
    PROBLEM: if we want either dispatch to general tuple, then #> can't return some kinded tuple.... we could have either-valid that 
    takes any tuple or validation-error or either and some other trick .. like parent kind, secondary kind, ... multiple kinds? 
    Depending on how these options behave in other cases or what they provide. We will decide this later with more examples.
    
## Kinds, implementation

Native kinds, preloaded by default are org.ryelang.core/num, org.ryelang.core/string, org.ryelang.core/block  etc.

We could define some other kinds, composites org.ryelang/person org.ryelang/user org.ryelang.email(content), etc ...

Everybody can define their own kinds and publish them online. Kinds have an url, validation rules, and optional converters.

Core kinds are preloaded all others must be defined, and mapped to local words:

    define-kinds {
      com.refaktor/v003 {
        user  // is named the same
        user-picture #pic
      }
      core/v1 {
        email
        iso-date #date
        http-header
      }
    }

PROBLEM:
Scripts / libraries that use those kinds still have to be loaded localy. Each script defiens it's own kinds ... so the interpreter needs to know in which script (kind mapping) (similar to namespace) current function is executing ... This is not what we planned for this stage.

SOLUTION?
Since these are universal identifiers, maybe interpreter can just load scripts directly and at each define mapping to current env integers for each unique kind, and at the same time replace all string KIND references to that specific integer, so at runtime there is no need to map local names to uri-kinds and know the locality .. it's all done in sequential fashion during the loading of scripts.
I THINK YES.


## Defininf local kinds

    kind #user { username: required string one-word name: string age: integer }
    kind #person { full-name: string { .capitalize-words } birth: date }
    conversion #user #person { name :full-name { now .year - age } :birth } // _ for asking a argument at conv time? :IDEA
    
    fns #user 'print { .get 'name |join* "Username: " .print }
    fns #person 'print { .get 'name |join* "Name: " .print }
    
    load-data >> #user .pass { .print } >> #person .print
    
    // or
    
    load-data >user> .pass { .print } >person> .print
    
    // pass executes block vith value as left and the block returns the same value
    Username: jimmycom
    Name: Jimmy Comute
    
    
    Star at the end of op-word or pipe-word makes second arg first, two stars would make third arg first. This makes sense, as first arg
    is special in composing a forward passing data flow.
    
          
          
          
## So what's the plan now

First figure out, if we lost any speed due to last changes (currying). 

Then make another branch and first add kind mapping logic.

Add another native and another user function that uses the kind information to dispatch. Let's still leave normal functions and normal natives there. Maybe we will neeed them still or more than we know. We can try converting core functions to dispatching ones, but normal can be a fallback still.

Make the kind mapper for the top .. that converst kinds to integers and with them loads the script. Kind url can now just be string at this point. Try to make some simple natives work and measure if we lost speed with it. 

Also make user functions with dispatch work and measure for speed.

If we have this working then we implement more language features. Like the validation dialect, the block autoargument option { .print } etc. Add kinds to all native types, etc ... convert functions.




