# crapmud_documentation

## API

The official client uses HTTP 1.1 GET requests, however any (1.1) request will work as the server does not distinguish, additionally your connection must be persistent to work correctly. Fill in the body for all below commands. The port is 6750

#### Experimental

There is a semi experimental websockets implementation available on 6760, use text mode. This will likely become the default in the future for custom clients as the HTTP implementation is slightly badly behaved

### Client -> Server

Every client command should be preceded by "client_command ". A correct format for a request is "client_command #fs.scripts.core()"

Calls which directly hook into the realtime chat system should be of the form "client_chat #hs.msg.send({channel:"\<YOURCHAN\>", msg:"\<YOURMSG\>"})"

The "client_chat " prefix simply suppresses script output from the server, so other commands could be piped through here if desired

The #up command follows the format client_command "#up scriptname \<SCRIPT_DATA\>"

Polling is performed by the request "client_poll" - this is the main driver that fetches chat so polls should be around ~1s in interval. This will probably be changed in the future because ddos'sing my own server is a poor move

The "client_command register client" command should be sent if no key.key file is present. The response is "command ####register secret <128bytekey>". This 128 byte key should be saved and used to auth, it is not retrievable from the server

The "client_command auth client <128bytekey>" command should be sent to auth the client after a new connection is made to the server, or on reconnect

In the event that your http lib dislikes binary, you can use register client_hex and auth client_hex to process hex instead. The format is little endian. If you ask for hex, the response will be "command ####register secret_hex <128bytekeyashex>"

### Server -> client

Responses from the server to "client_command "s are of the form "command \<RESPONSE\>"

The full list of commands that will provoke a valid response are user <username>, #up, #dry, #remove, #public, #private, register client, auth_client, auth_client_hex and a JS command (which is any text which is not one of the former)

Responses to "client_poll"'s go as following: "chat_api sizeof_next |num_channels \<chan0 chan1 chan2...\> |<<<sizeof_next2  |channel raw_chat_string|>>>

The sizeof_next items refer to the size of the elements surrounded by |s, in bytes. The first sizeof_next's is the size of |num_channels \<channels\> |, and the second is |channel raw_chat_string|

The section in <<<>>> repeats fully. The section in \<\> is unbounded and may contain any number of entries (ie num channels in this case)

The server sends no response for a "client_chat " command if you use the websocket endpoint. Responses from the server should be stashed in a file somewhere, and reloaded next script run

#### Experimental

There is an experimental client_poll_json mode. Responses to client_poll_json are in the format "chat_api_json JSON". The JSON format is: {"channels":[channel_list], "data":[{"channel":channel, "text":raw_chat_string}]}, where array items may repeat indefinitely

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

\#hs.msg.send({channel:"name", msg:"message"})

\#ms.msg.recent({channel:"name"}) -> channel defaults to "0000", takes optional {count:number} or {array:1}

\#ms.msg.manage -> takes 1 of join:channel, create:channel, leave:channel

\#ns.users.me -> takes optional {array:1} argument

\#ns.items.create({type:number}) -> type goes from 0 to 7, cheat to hack in items for testing. Type 1 is locks, use lock_type:name

\#ls.items.xfer_to({to:"username", idx:item_index})

\#ms.items.manage -> takes optional {array:1} or {full:1} (which displays detailed info), or takes optional {load:item_index} or {unload:item_index} 

\#ls.nodes.manage({load:index}) -> takes load/unload index of a lock_type

\#ls.nodes.port() -> returns your port/loc


### DB

\#db.f({example:"query"}) -> returns a cursor, use .array() or .first() to get the results

\#db.u({doot:{$exists:true}}, {$set:{doot:"nope"}}) -> finds all documents which have a key doot, and sets their key doot to 'nope'

\#db.i({key:"something", key2:"somethingelse"}) -> inserts a new document

### Other

#up scriptname -> will upload a script to the server

#dry scriptname -> will test run a script upload to the server but not commit the changes

#private scriptname -> will make a script private

#public scriptname -> will make a script public

#remove scriptname -> will remove a script from the server

\# -> lists all local scripts for that user

\#dir -> opens the script directory (shared between users)

\#edit scriptname -> creates or opens a script for editing

### Calling Scripts from a String

While \#ns.script.name() is the most straightforward way to call a hardcoded script, its also possible to call a script from a string or variable

When you call \#ns.script.name(), it expands to ns_call("script.name")(). ns_call("script.name") returns a function object and does not call the function itself, so you may do var x = ns_call("script.name"); x()

Currently, the fs/hs/ms/ls/ns_call functions only get injected into your code (for security reasons) when at least 1 \#fs/hs/ms/ls/ns.script.name() call is found

For example, if you call \#ms.script.name(), ms_call, hs_call and fs_call are available in your script

Example:

	function(c, a)
	{
		#ms.invalid.script; //does not call the script, but is a valid statement to the parser
	
		//you may not use ls_call() here
	
		return hs_call("i20k.highsec")(); //dont forget the second set of ()s!
	}

## GENERAL

The console prompt you're given is a raw JS terminal, essentially of the form "return \<input\>". This means that if you enter 1+1, you get 2

Scripts are run through eg #fs.script.name(), or #script.name() which maps to nullsec

### The Sandbox

Due to the way its implemented, there are no restrictions on the sandbox, you should not be able to edit the global object in a meaningful way

If you are able to mess with any of the global properties in a way that carries over into another script, let me know