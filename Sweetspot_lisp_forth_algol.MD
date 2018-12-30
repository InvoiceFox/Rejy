# Looking at factor, lisps, rebol, algol

> This could be horribly pretentious text. I am aware of that, you reader - please be aware of that too.
> I am nobody, but I can still type, and you can ignore ...

### Factor - a stack based language

**PRO:** these languages reject **variable names**. And many times variables are just unnecesary noise that breaks the flow on logic. This is some factor code:

    "Hello, world" print                                            ; prints "Hello, world"
    
    { 4 8 15 } [ 2 * ] map .                                        ; -- puts { 8 16 30 } on stack
    
    "A man, a plan, a canal: Panama." [ Letter? ] filter >lower     ; -- puts amanaplanacanalpanama on stack
    
    "http://factorcode.org" http-get nip string>xml                 ; loads page, turns it into XML nodes
    "a" deep-tags-named                                             ; finds all a tags
    [ "href" attr ] map                                             ; maps list of tags to list of href attrs
    [ print ] each                                                  ; prints each of links

**CONS:** the language introduces stack shuffling words, which is even bigger line noise.

**PRO:** because of this, many advanced solutions vere invented that make the code nicer flowing (see factorcode.org)

**PRO:** the postfix sytax is naturaly oriented as the operations flows in a program

From the document: http://factorcode.org/littledan/dls.pdf

> Code written in this style, in which a single input value
> is gradually transformed and reshaped in distinct steps into
> a result, is known as pipeline code, named due to the resem-
> blance to the use of pipes in Unix shells. Pipeline code is
> expressed very naturally in Factor; given several words, say
> a, b, c, each taking a single object from the stack and push-
> ing a single result, the code  that applies each word to the
> result of the previous word is simply written as:

    10 a b c            ; -- factor code
    
    c(b(a(10)))         ; -- python code
    
    (c (b (a 10)))      ; -- lisp code
    
    c b a 10            ; -- rebol code
    
    c b a 10            ; -- Rejy code
    10 |a |b |c         ; -- Rejy code using pipewords

Rejy learns from all this and offers few features.



### LISPS - homoiconic list based languages



    
