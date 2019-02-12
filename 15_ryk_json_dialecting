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
    
    ryk -j "{ 'all_data 'people_data |filter 'age > 25 |forall [ print 'name ] }' ... litword also works as eval expression
                                                                                  ... this would be replicated in context of where
                                                                                  
                                                                                  
Where example

  with person {
    join 'name 'surname
  }
  
  ; with takes block and insers the 'get object' in front of every lit-word
                                                                                  
  'with in this case would be loader word ... similar to factor's .. hm .. so we hove sort of macros .. the dialect above is just 
  recursive with ...
