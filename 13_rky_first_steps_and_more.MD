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

    cat people.csv | ./ryk "start { j: json-array } { o: json-obj , set o 'name _1 , set o 'surname _2 , set o 'birth now/year - age , add j o } end { j }"
    
The simple manual way above. The , as a expression guard looks really usefull and nice. Otherwise expressions above would be very hard to comprehend and two spaces don't help much.

## Making it nicer

    cat people.csv | ./ryk "
    start { j: json-array } 
    { add j json-obj-from csv-line _ '; [ 'name 'surname <birth> { now/year - _3 } ] } 
    end { j }"

more structured and declarative, but also more complex. The main problem I see in this code is hidden nesting which makes it quite incomprehensable.

    cat people.csv | ./ryk "
    start { j: json-array } {
    add j (json-obj-from (csv-line _ ',) [ 'name 'surname <birth> { now/year - _3 } ]) } 
    end { j }"

opwords probably wouldn't help much with clarity here... and because of left to right execution this wouldn't work. 

    cat people.csv | ./ryk "
    start { j: json-array } 
    { j *add (_ *csv-line ',) *json-obj-from [ 'name 'surname <birth> { now/year - _3 } ]) } 
    end { j }"

Does turning of arguments help us any? Let's say that ** switches the first two arguments around

    cat people.csv | ./ryk "
      start { j: json-array } 
      { (_ *csv-line ',) *json-obj-from [ 'name 'surname <birth> { now/year - _3 } ] **add j   } 
      end { j }"
      
Would stack help?
      
    cat people.csv | ./ryk "
     start { j: json-array } 
     { csv-line _ '; |push , json-obj-from pop [ 'name 'surname <birth> { now/year - _3 } |push , j +add pop  ] } 
     end { j }"

Temporary variables?
     
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
    
    
