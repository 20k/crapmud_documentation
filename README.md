# crapmud_documentation

## API

All requests are get requests

### Client -> Server

Every client command should be preceded by "client_command ". A correct format for a request is "client_command #fs.scripts.trust()"

Calls which directly hook into the realtime chat system should be of the form "client_chat #hs.msgs.send({channel:"\<YOURCHAN\>", msg:"\<YOURMSG\>"})"

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

\#hs.cash.balance

\#ms.cash.xfer_to({to:"username", amount:number})

\#fs.cash.xfer_to_caller({amount:number})

\#fs.scripts.get_level({name:"user.scriptname"})

\#fs.scripts.core

\#ms.scripts.me

\#fs.scripts.public -> takes optional {array:1} or optional {sec:number}

\#hs.msgs.send({channel:"name", msg:"message"})

\#ms.msgs.recent({channel:"name"}) -> channel defaults to "0000", takes optional {count:number} or {array:1}

\#ns.users.me -> takes optional {array:1} argument

\#ns.items.create({type:number}) -> type goes from 0 to 6, cheat to hack in items for testing

\#ls.items.xfer_to({to:"username", idx:item_index})

\#ms.items.manage -> takes optional {array:1} or {full:1} (which displays detailed info), or takes optional {load:item_index} or {unload:item_index} 

\#db.f({example:"query"}) -> returns a cursor, use .array() or .first() to get the results

\#db.u({doot:{$exists:true}}, {$set:{doot:"nope"}}) -> finds all documents which have a key doot, and sets their key doot to 'nope'

\#db.i({key:"something", key2:"somethingelse"}) -> inserts a new document

### Other

#up scriptname -> will upload a script to the server

#private scriptname -> will make a script private

#public scriptname -> will make a script public

#remove scriptname -> will remove a script from the server

\# -> lists all local scripts for that user

\#dir -> opens the script directory (shared between users)

\#edit scriptname -> creates or opens a script for editing

## GENERAL

The console prompt you're given is a raw JS terminal, essentially of the form "return \<input\>". This means that if you enter 1+1, you get 2

Scripts are run through eg #fs.script.name(), or #script.name() which maps to nullsec

### The Sandbox

Due to the way its implemented, there are no restrictions on the sandbox, you should not be able to edit the global object in a meaningful way

If you are able to mess with any of the global properties in a way that carries over into another script, let me know