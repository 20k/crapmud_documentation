# crapmud_documentation

## API

The official client uses websockets on 6760, the server will respond back to you in whichever mode you pick (text/binary), however be aware that commands that respond in binary will not work in text mode

As the websocket mode is new, there may be bugs, please let me know if you find any

#### Deprecated

There is a deprecated HTTP implementation on 6750 that will be removed in a future update

### Client -> Server

Every client command should be preceded by "client_command ". A correct format for a request is "client_command #fs.scripts.core()"

Calls which directly hook into the realtime chat system should be of the form "client_chat #hs.msg.send({channel:"\<YOURCHAN\>", msg:"\<YOURMSG\>"})"

The "client_chat " prefix simply suppresses script output from the server, so other commands could be piped through here if desired

The #up command follows the format client_command "#up scriptname \<SCRIPT_DATA\>"

Polling is performed by the request "client_poll" - this is the main driver that fetches chat so polls should be around ~1s in interval. This will probably be changed in the future because ddos'sing my own server is a poor move

The "client_command register client" command should be sent if no key.key file is present. The response is "command ####register secret <128bytekey>". This 128 byte key should be saved and used to auth, it is not retrievable from the server

The "client_command auth client <128bytekey>" command should be sent to auth the client after a new connection is made to the server, or on reconnect

In the event that your http lib dislikes binary, you can use register client_hex and auth client_hex to process hex instead. The format is little endian. If you ask for hex, the response will be "command ####register secret_hex <128bytekeyashex>"

#### Autocompletes

You may request an autocomplete from the server with the format "client_scriptargs script.name". Additionally, to request the experimental JSON mode use "client_scriptargs_json script.name"

There is currently no way to batch autocompletes together, however with websockets being the default, this isn't so much of an issue

### Server -> client

Responses from the server to "client_command "s are of the form "command \<RESPONSE\>"

The full list of commands that will provoke a valid response are user <username>, #up, #dry, #remove, #public, #private, register client, auth_client, auth_client_hex and a JS command (which is any text which is not one of the former)

Responses to "client_poll"'s go as following: "chat_api sizeof_next |num_channels \<chan0 chan1 chan2...\> |<<<sizeof_next2  |channel raw_chat_string|>>>

The sizeof_next items refer to the size of the elements surrounded by |s, in bytes. The first sizeof_next's is the size of |num_channels \<channels\> |, and the second is |channel raw_chat_string|

The section in <<<>>> repeats fully. The section in \<\> is unbounded and may contain any number of entries (ie num channels in this case)

The server sends no response for a "client_chat " command if you use the websocket endpoint. Responses from the server should be stashed in a file somewhere, and reloaded next script run

#### Autocompletes

The response for autocompletes is "server_scriptargs sizeof_next |script.name| <sizeof_next |key| sizeof_next |val| >". The section in <> repeats fully

In the event a script does not exist, or is a bad scriptname, the response is "server_scriptargs_invalid script.name". In the event that the request is unintelligable, the response is "server_scriptargs_invalid"

In the event you are being ratelimited, you will receive "server_scriptargs_ratelimit script.name", or "server_scriptargs_ratelimit_json JSON", where the JSON format is {script:"name"}

#### Experimental: client_poll_json

There is an experimental client_poll_json mode. Responses to client_poll_json are in the format "chat_api_json JSON". The JSON format is: {"channels":[channel_list], "data":[{"channel":channel, "text":raw_chat_string}]}, where array items may repeat indefinitely

#### Experimental: client_scriptargs_json:

There is an experimental client_scriptargs_json mode. Responses to client_scriptargs_json are in the format "server_scriptargs_json JSON". The JSON format is: {"script":"scriptname", "keys":["key_1", "key_2"], "vals":["val_1, val_2"]}, where array items may repeat indefinitely

## SCRIPTING

All scripts should go in the ./scripts folder. Scripts should be named "user.scriptname.js", and can be uploaded with #up scriptname. EG if I am logged in as user "example", I make example.hello_world.js in ./scripts, and #up hello_world

Scripts follow the format

	function(context, args)
	{
	
	}

### Implemented scripts

\#hs.cash.balance

\#ms.cash.xfer_to({to:"user", amount:number})

\#fs.cash.xfer_to_caller({amount:number})

\#fs.cash.steal({from:"user", amount:number}) -> steals cash from a target with a breached breach node

\#fs.scripts.get_level({name:"user.scriptname"})

\#fs.scripts.core -> takes optional {array:1} parameter

\#ms.scripts.me -> takes optional {array:1} parameter

\#fs.scripts.public -> takes optional {array:1} or optional {sec:number}

\#hs.msg.send({channel:"name", msg:"message"})

\#ms.msg.recent({channel:"name"}) -> channel defaults to "0000", takes optional {count:number} or {array:1}

\#ms.msg.manage -> takes 1 of join:channel, create:channel, leave:channel

\#ns.users.me -> takes optional {array:1} argument

\#ns.items.create({type:number}) -> type goes from 0 to 7, cheat to hack in items for testing. Type 1 is locks, use lock_type:name

\#ls.items.xfer_to({to:"user", idx:item_index})

\#ms.items.manage -> takes optional {array:1} or {full:1} (which displays detailed info), or takes optional {load:item_index} or {unload:item_index} 

\#ls.nodes.manage -> displays nodes and attached locks. Is no longer used for equipping locks, use #items.manage

\#ls.nodes.port -> returns your port/loc

### Autocompletes

You may specify a scripts autocompletes like #autos(test:1, test2:"hello", test:3:"bitconnect"). The parser isn't that fancy so you may end up with slightly incorrect behaviour if you do eg #autos(test:1, test2:"test1:")

### DB

\#db.f({example:"query"}) -> returns a cursor, use .array() or .first() to get the results. Takes an optional second projection argument

\#db.u({doot:{$exists:true}}, {$set:{doot:"nope"}}) -> finds all documents which have a key doot, and sets their key doot to 'nope'

\#db.i({key:"something", key2:"somethingelse"}) -> inserts a new document

### Other (non scriptable)

\#up scriptname -> will upload a script to the server

\#dry scriptname -> will test run a script upload to the server but not commit the changes

\#private scriptname -> will make a script private

\#public scriptname -> will make a script public

\#remove scriptname -> will remove a script from the server

\# -> lists all local scripts for that user

\#dir -> opens the script directory (shared between users)

\#edit scriptname -> creates or opens a script for editing

\#clear_autos -> clears autocompletes

\#shutdown -> shuts down the client

user \<username\> -> changes user, create automatically (will be changed in a future update)

### Calling Scripts from a String

While \#ns.script.name() is the most straightforward way to call a hardcoded script, its also possible to call a script from a string or variable

When you call \#ns.script.name(), it expands to ns_call("script.name")(). ns_call("script.name") returns a function object and does not call the function itself, so you may do var x = ns_call("script.name"); x()

Currently, the fs/hs/ms/ls/ns_call functions only get injected into your code (for security reasons) when at least 1 \#fs/hs/ms/ls/ns.script.name() call is found. Additionally, the parser checks for *_call directives, and will inject the correct functions on detecting eg hs_call

For example, if you call \#ms.script.name(), ms_call, hs_call and fs_call are available in your script. If you call ls_call, ls_call, ms_call, hs_call, and fs_call are available

Example:

    ///this script is highsec due to hs_call
	function(c, a)
	{
		return hs_call("i20k.highsec")(); //dont forget the second set of ()s!
	}

## GENERAL

The console prompt you're given is a raw JS terminal, essentially of the form "return \<input\>". This means that if you enter 1+1, you get 2

Scripts are run through eg #fs.script.name(), or #script.name() which maps to nullsec

### The Sandbox

Due to the way its implemented, there are no restrictions on the sandbox, you should not be able to edit the global object in a meaningful way

If you are able to mess with any of the global properties in a way that carries over into another script, let me know