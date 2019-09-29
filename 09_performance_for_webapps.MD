# Why I think Rejy will have adequate performance for web-apps

Rejy, being similarly to AST interpreted won't be that fast. So why do I think it will still perform OK for web-apps. All in all,
webapps are usually IO bound, not CPU bound. The process is waiting for database or TCP communication most of the time. But still:

 * Webapp code (in my webapps) executes only single limited set of commands per requst. In 99% cases it never(1) loops: 
   * It just validates inputs. 
   * Generates SQL and sends it to database
   * Serializes result in common format (like JSON) (1) here it does loop, but it's part of protocol, it's always uniform
 
 * My webapp needs to have 
   * **good networking support** 
   * **good database drivers**
   * **fast serialisation**
   All these will be native Go, so they will be Go-like fast. Even if Rejy will be slower than Rebol was, the request will be
   served much much faster - mainly because of much faster serialisation.
 
 * The only time when Rebol now loops (and is slow) is serialisation, and becomes a problem when you are trying to generate 
    1000s of rows/objects with few 10s of fields into one big string. This is needed to generate a response to a web client, or to 
    generate payload to send to PDF or Excel generating server. I tested and string concatenation was like 10x faster in nodejs 
    Rejy than rebol. I expect that Go version will be even faster and all serialisations will be written directly in Go this time.
    
  * Even the validation and SQL generating dialects could be written and compiled in Go. Even though they "don't loop"
  
  
