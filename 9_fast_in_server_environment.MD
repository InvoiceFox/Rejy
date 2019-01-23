# Why I thing Rejy Go will be more than fast enough for writing webapps

Rejy, being similarly to AST interpreted won't be that fast. So why do I think it will behave great on for web-apps:

 * Webapp code (in my webapps) executes only single limited set of commands per requst. In 99% cases it never(1) loops: 
   * It just validates inputs. 
   * Generates SQL and sends it to database
   * Serializes result in common format (like JSON) (1) here it does loop, but it's part of protocol, it's always uniform
 * My webapp needs to have **good networking support** and good database drivers (which will be native Go), serialisation will be
   written in Go so it's compile-like fast.
 * The only time when Rebol now loops (and is slow) is serialisation, and becomes a problem when you are trying to generate 
   1000s of rows/objects into big string. It might need serialisation to generate a response to client, or to generate payload
   to send to PDF or Excel generating server. I tested and string concatenation is like 10x faster in nodejs Rejy than rebol.
   I expect that Go version will be even faster and all serialisations will be written directly in Go
