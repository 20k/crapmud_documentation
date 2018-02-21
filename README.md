# crapmud_documentation

## API

All requests are get requests

### Client -> Server

Every client command should be preceded by "client_command ". A correct format for a request is "client_command #fs.scripts.trust()"

Calls which directly hook into the realtime chat system should be of the form "client_chat #hs.chats.send({channel:"\<YOURCHAN\>", msg:"\<YOURMSG\>"})"

The "client_chat " prefix simply suppresses script output from the server, so other commands could be piped through here if desired

The #up command follows the format client_command "#up scriptname \<SCRIPT_DATA\>"

Polling is performed by the request "client_poll" - this is the main driver that fetches chat so polls should be around ~1s in interval. This will probably be changed in the future because ddos'sing my own server is a poor move

The "client_command register client" command should be sent if no key.key file is present

### Server -> client

Responses from the server to "client_command "s are of the form "command \<RESPONSE\>"

Responses to "client_poll"s from the client are of the form "chat_api channel \<DATA\>". Data is formatted text with colour codes and should be displayed vanilla

The server sends no response for a "client_chat " command. Responses from the server should be stashed in a file somewhere, and reloaded next script run

## SCRIPTING

All scripts should go in the ./scripts folder. Scripts should be named "user.scriptname.js", and can be uploaded with #up scriptname. EG if I am logged in as user "example", I make example.hello_world.js in ./scripts, and #up hello_world

Scripts follow the format

function(context, args)

{
	
	
	
}

### Implemented scripts

\#hs.accts.balance

\#fs.scripts.get_level

\#ms.accts.xfer_gc_to

\#fs.scripts.trust

\#fs.accts.xfer_gc_to_caller

\#ms.scripts.user

\#hs.chats.send({channel:"\<string\>", msg:"\<msg\>"})

\#ms.chats.recent({channel:"0000_by_default", count:num_default_1, pretty:1}) -> pretty:0 returns an array, pretty:1 returns exactly what the server pipes you with chat messages

\#db.f({example:"query"}) -> returns a cursor, use .array() to get the results

\#db.u({doot:{$exists:true}}, {$set:{doot:"nope"}}) -> finds all documents which have a key doot, and sets their key doot to 'nope'

\#db.i({key:"something", key2:"somethingelse"}) -> inserts a new document

## GENERAL

The console prompt you're given is a raw JS terminal, essentially of the form "return \<input\>". This means that if you enter 1+1, you get 2

Scripts are run through eg #fs.script.name(), or #s.script.name() which maps to nullsec

### The Sandbox

Due to the way its implemented, there are no restrictions on the sandbox, you should not be able to edit the global object in a meaningful way

If you are able to mess with any of the global properties in a way that carries over into another script, let me know