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

