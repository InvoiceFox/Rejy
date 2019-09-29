## The Spruce idea

### // 27.9.2019 

Started adding SPRUCE LANG SUPPORT.

I started making a spruce/build parser. It's a indentation aware tree representation parser.
I created a SprNode ... a node in a tree that holds a Rye Object (word, string, block), list of child SprNodes, depth
and a parent SprNode.
I added a function to find the right parent to add a node to based on a last parent and depth.

Next. I should add the Rye loader, that turns strings to Rye Objects (word, string, block). And make a unit test that parses
fist tree.

    when
     email "help string"
      is 
       received 
        do
         {todo:block}
         [ on-event 'email-received { 'it 'email } todo ]
       sent
        do
         {todo:block}
         [ ]
       blocked
        do
         {todo:block}
         [ ]     
    if
     {cond:boolean}
      do
       {todo:block}
       [ core/if cond todo ]
    it's
     {key:string}
     [ core/at it key ]
    {a:boolean}
     and
      {b:boolean}
      [ core/and a b ]
    {val:string}
     includes
      text
       {text:string}
     is
      from
       domain
        {domain:string}
	
I also added the tests file to loader and it basically works.

**NEXT**: given that we have a basic parser and with it a basic tree, we can make a simple walker / executor next.
think about making proposer too and how we could add tree branches to existing tree ... etc

how could we make a simple console based IDE ... or emacs based IDE? How do IDE-s connect to language runtimes at all.

### //29.9.2019 #SPRUCE 2

Ok, so basic expression works. We would next need to add argument definition and pickup, but to do it as example above we
would need to extend the parser with these forms {name:type}. Are there any other forms that we already parse for?

example option #1:

	add:
	 numbers
	  {a:number}
	   and
	    {b:number}
	     { add a b }
	
	Sum: add numbers 10 and 5 
	. returns 15

could we remove the {}, since they are already used for blocks too?

	add
	 nubmers
	  first:number
	   and
	    second:number
		 { add a b }
	
Not enough visual distinction so looks cleaner, but the tree looks more complex, because
args are not distincted apart imediatelly. We also might need to pack in more logic in this
pickup words, so just : is not distincion enough. We might pick up multiple types, add transformers, 
composers etc, so a whole mini language might get in later.

	my:
	 number
	  { 041741612 }

	in:
	 high
	  alert
	   mode
	    { get-alert-mode .equal? "H" }

	when:
	 pinged
	  if
	   {cond:boolean}
	    do
		 {code:block}
		  { if cond { do/spr code }}

	time:
	 is
	  between
	   {t1:time}
	    and
		 {t2:time}
		  { get-time .between t1 t2 }

	when pinged if in high alert mode and time is between 20:30 and 06:30 do {
		sms my number with text "something weird is going on"
	}

	alert:
	 numbers 
	  { { 041741612 041741623 } }

	for:
	 all
	  {list:block}
	   do
	    {code:block}
		 { for-all 'it list { do/spr bind [ 'It it ] code } }

	when pinged if in high alert mode and time is between 20:30 and 06:30 do {
		for all alert numbers do { 
			sms It with text "something weird is going on"
		}
	}

...

So, first we need to add those argument placeholders {name:type} to Rye parser. 
Then we can use them inside Spruce builer and user quite easily.

Then we need to decide, where the varables picked up get stored and for how 
long. It seems obvious that each start of spruce expression should have it's own
namespace, and namespace is used in code, if there is one. After the end of 
the branch that namespace is discarded.

There could later show to be some special forms to directly set It, or decompose
stuff. Or compose to the It object for example.

##composition of object example?

	create:
	 person
	  {It:object:create}
	  with *
	   name
	    {name:string->It}
		 *
	   surname
	    {surname:string->It}
		 *
	   age
	    {age:number->It}
	 *

general object could be a Rye tuple instead, that has defined validation rules, and those
are checked and returned as error if not meet when used



	create person with name "Janko"
	. returns Validation error, tuple Person { surname: required }

## decomposition of tuple example?

	get:
	 age
	  of
	   {p:person}
	    { at p 'age }
	
or potential automatic decomposition inside the arg block

	get
	 age
	  of
	   {p:person:'age->}
	   
not sure if it's the right form, or worth it ... maybe it will show to be, since we do get rid od one whole block. But we will
jsut later see, what internal dialects could be usefull and the visual language of them. Maybe the pickup rule should have validation dialect also possible, and manipulation dialect, transformation dialect ... maybe custom spruce or rye code?

	get
	 age
	  of
	   {p:person'birth-date->calc-age->}

or if we use some already known Rye's language

	get
	 age
	  of
	   {p:person .get 'birth-date .calc-age }
	   
yes, this seems the way!

maybe blocks of code with arguments in general could be just these in Rye too? ... we miss short anonymous function syntax now

	{a,b add a b }
	{pers:person .get 'name .join* "Hi, I'm " }

Then the composition could get:

	create:
	 person
	  {It:object:new}
	  with *
	   name
	    {name:string .set It}
		 *
	   surname
	    {surname:string .set It}
		 *
	   age
	    {age:number .set It}
	 *
	 
Where name:string would create some selfavare wariable, that knows it's own name and value. so set would accept name and value from it and use the name to set the filed name in object It. Hm ... interesting weird ideas are just lying around it seems. If they are really usefull is another question.


