## JavaScript runtime implementation

### Internal representation of values

First all values and nodes were boxed. To get leaner performance out of JavaScript we now represent all native values as native JS values. All words are indexed and words themselves require just integer type and index of a word, that is now Uint16Array(), which I think is very optimal. Rejy objects are JS objects, with field __tag__ for tag.

Arrays where first element is Uint8array() are still unindexed words or complex types. After they get indexed / preprocessed they turn to Uint16array or Array with first element Uint16array (like func, generic and tag definitions).

We need to test if this array options are faster than objects with key/value paids for complex object/tag/generic definitions.

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
if we visit array we check if first item is Uint8Array() if it is it's composite type. If it is we turn it into Uint16Array(2) where first value is type and second is index in process wide WORDSINDEX.
