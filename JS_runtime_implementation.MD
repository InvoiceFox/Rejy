## JavaScript runtime implementation

### Representation of Rejy nodes in JS

Currently, each value in Rejy is small JS Array. First value is integer type, second is value, third and so on are optional.
```
{ word: 2 { janko } } // turns to

[ 1, // block
   [ 13, "word" ], // setword
   [ 101, 2 ],     // 
   [ 1,
      [ 11, "dog" ],
      [ 14, "woof" ]
   ]
]
```

### Internal representation of values

Problem: Some of functions we have now in Rejy work directly on unboxed JS values, for speed and simplicity purposes, but the AST Nodes aren't like that. 
* Solution would be that we would use direct JS values for Native types in AST. String, Number, Int .. what about arrays and objects? ... and are there any benefits of these two not being different syntactically in Rejy too? Object is not really series and functions on series don't really make sense on it or even work ... ? _ ? Minus is that then we don't have only series there
* Another solutio is that we transfer from AST to interpreter values and back (get AST, turn to array/object, manipulate in JS and return as AST Nodes again)
* Don't know what makes more sense ... maybe we try both... 
* It makes sense that the series as code is series, but  function turns it into object (they series functions make no sense any more - before that .. when it's still code/data series operations make full sense)

### Optimisation - native JS values instead of arrays like above
```
[ ] is block
"" is string
213 or 21.42 is number

// words
new Uint16Array([11, <idx>]); word
new Uint16Array([12, <idx>]); setword 
new Uint16Array([13, <idx>]); getword
new Uint16Array([14, <idx>]); litword
new Uint16Array([15, <idx>]); opword
new Uint16Array([16, <idx>]); tagword
new Uint16Array([17, <idx>]); settagword

// object, basic implementation
{ __tag__: Uint16array([1, <idx>]), prop1: <val1>, ...  } 
// if there will be multiple tags, first tag will be number of tags or 2

// tag definition
[ new Uint16array([101]), [ ... block of tag definition dialect (validation dialect and whats needed) ... ] ]

// function definition
[ new Uint16array([102]), [ ... argument block ... ] [ ... function body block ... ]]
```

```
{ word 2 { "dog" 'woof } } // is now 
[ // block
   [ { "0": 11 }, "word" ], // Uint8Array(11) is word
   2, // nuber
   [ // block
      "dog", // string
      [ { "0": 14 }, "woof" ] // Uint8Array(14) is litword
   ]
]
```
if we visit array we check if first item is Uint8Array() if it is it's composite type. If it is we turn it into Uint16Array(2) where first value is type and second is index in process wide WORDSINDEX
