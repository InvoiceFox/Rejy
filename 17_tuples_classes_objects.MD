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
