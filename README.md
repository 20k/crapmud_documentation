# net_code documentation

## API

The official client uses websockets on 6760, the server will respond back to you in whichever mode you pick (text/binary), however be aware that commands that respond in binary will not work in text mode

As the websocket mode is new, there may be bugs, please let me know if you find any

#### Deprecated

There is a deprecated HTTP implementation on 6750 that will be removed in a future update

client_poll and client_scriptargs are both deprecated and now undocumented. Use the JSON versions instead

### Client -> Server

Every client command should be preceded by "client_command ". A correct format for a request is "client_command #fs.scripts.core()"

Calls which directly hook into the realtime chat system should be of the form "client_chat #hs.msg.send({channel:"\<YOURCHAN\>", msg:"\<YOURMSG\>"})"

The "client_chat " prefix simply suppresses script output from the server, so other commands could be piped through here if desired

The #up command follows the format client_command "#up scriptname \<SCRIPT_DATA\>". Additionally, you may use #up_es6 to request es6/typescript compilation, which is slower

Polling is performed by the request "client_poll_json" - this is the main driver that fetches chat so polls should be around ~1s in interval. This will probably be changed in the future because ddos'sing my own server is a poor move

The "client_command register client" command should be sent if no key.key file is present. The response is "command ####register secret <128bytekey>". This 128 byte key should be saved and used to auth, it is not retrievable from the server

The "client_command auth client <128bytekey>" command should be sent to auth the client after a new connection is made to the server, or on reconnect

In the event that your http lib dislikes binary, you can use register client_hex and auth client_hex to process hex instead. The format is little endian. If you ask for hex, the response will be "command ####register secret_hex <128bytekeyashex>"

The "client_terminate_scripts JSON" command can be sent to terminate a realtime script. The JSON format is {"id":id}. If the id is -1, it will terminate any realtime script

The "client_script_keystrokes JSON" command can be sent to send keystrokes. The JSON format is {"id":id, "keys":"asdfqwer\n2134"}

#### Autocompletes

You may request an autocomplete from the server with the format "client_scriptargs_json script.name"

There is currently no way to batch autocompletes together, however with websockets being the default, this isn't so much of an issue

### Server -> client

Responses from the server to "client_command "s are of the form "command \<RESPONSE\>". Be aware that server responses generally may include colour codes EG `Dhello`, that you will be required to parse yourself

The full list of commands that will provoke a valid response are user <username>, #up, #dry, #remove, #public, #private, register client, auth client, auth client_hex and a JS command (which is any text which is not one of the former)

Responses to client_poll_json are in the format "chat_api_json JSON". The JSON format is: {"channels":[channel_list], "data":[{"channel":channel, "text":raw_chat_string}], "tells":[{"user":user, "text":raw_chat_string}]}, where array items may repeat indefinitely

The server sends no response for a "client_chat " command if you use the websocket endpoint. Responses from the server should be stashed in a file somewhere, and reloaded next script run

Async updates are sent to the client in the format "command_realtime_json JSON". The JSON format is {"id":id, "msg":msg}. The id is globally unique across every possible script invocation, and uniquely identifies one script run

#### Autocompletes

Responses to client_scriptargs_json are in the format "server_scriptargs_json JSON". The JSON format is: {"script":"scriptname", "keys":["key_1", "key_2"], "vals":["val_1, val_2"]}, where array items may repeat indefinitely

In the event a script does not exist, or is a bad scriptname, the response is "server_scriptargs_invalid_json JSON", where the JSON format is {"script":"scriptname"}. In the event that the request is unintelligable, the response is "server_scriptargs_invalid_json"

In the event you are being ratelimited, you will receive "server_scriptargs_ratelimit_json JSON", where the JSON format is {"script":"scriptname"}

## SCRIPTING

All scripts should go in the ./scripts folder. Scripts should be named "user.scriptname.js", and can be uploaded with #up scriptname. EG if I am logged in as user "example", I make example.hello_world.js in ./scripts, and #up hello_world

Scripts follow the format

	function(context, args)
	{
	
	}

### Implemented scripts

\#hs.cash.balance

\#ms.cash.xfer_to({user:"user", amount:number})

\#fs.cash.xfer_to_caller({amount:number})

\#fs.cash.expose({user:"user"}) -> shows you the amount of cash a target has

\#fs.cash.steal({user:"user", amount:number}) -> steals cash from a target with a breached breach node

\#fs.scripts.get_level({name:"user.scriptname"})

\#fs.scripts.core -> takes optional {array:1} parameter

\#ms.scripts.me -> takes optional {array:1} parameter

\#fs.scripts.public -> takes optional {array:1} or optional {sec:number}

\#hs.msg.send({channel:"name", msg:"message"})

\#hs.msg.tell({user:"name", msg:"hello"}) -> sends a private message to a user

\#ms.msg.recent({channel:"name"}) -> channel defaults to "0000", takes optional {count:number} or {array:1}. Additionally takes {tell:true} to retrieve tells instead

\#ms.msg.manage -> takes 1 of join:channel, create:channel, leave:channel

\#ns.users.me -> takes optional {array:1} argument

\#fs.item.steal({user:"target", idx:0}) -> steals the user's item at idx:id, costs 10 cash which you can confirm with {confirm:true}

\#fs.item.expose({user:"target"}) -> lists a users current items if they are breached

\#ls.item.xfer_to({user:"user", idx:item_index})

\#ms.item.manage -> takes optional {array:1} or {full:1} (which displays detailed info), or takes optional {load:item_index} or {unload:item_index} 

\#ls.item.bundle_script({name:"scriptname", idx:0, tag:"shrt_tag"}) -> inserts the source of a script (host.scriptname) into a bundle at idx:id. Tag modifies the item name, must be 8 characters or less

\#ls.item.cull({idx:0}) -> unloads and deletes the item at idx:id

\#ns.item.register_bundle({name:"arbitraryname", idx:0}) -> registers a bundle at idx:id to be run as host.arbitraryname()

\#ls.nodes.manage -> displays nodes and attached locks. Is no longer used for equipping locks, use #items.manage

\#ls.nodes.view_log({user:"user", NID:id}) -> takes optional {array:1}. Must have a clear breach path to the node in question

\#ls.net.view({user:"name"}) -> views the network connections associated with a user or npc

\#ls.net.map({user:"name", n:7}) -> views the network connections associated with a user or npc in 2d, with a max depth of n

\#ns.net.access({user:"name"}) -> access an NPC's command interface. Takes {add_user:"name"} or {remove_user:"name"} or {view_users:true}. Costs 200 cash to execute a command on an npc you have not added yourself to, in which case you must pass {confirm:true} to confirm payment

\#fs.net.hack({user:"name", extra_args:"example"}) -> hacks a user at user:"name", passes args forwards

\#ns.net.switch({user:"name"}) -> switches your terminal input to be run through this npc instead. Currently only works for npcs, and your main user that you are running on (from the user <username>) command

### Realtime Scripting

To set a script to be realtime, use

`set_is_realtime_script();`

The script then is set into realtime mode. There are 3 callbacks that you can return:

on_draw, on_update(dt), and on_input(char)

For every 16ms you get 2ms of processing. You can terminate a realtime script with control-c, or by closing the associated window

To exit realtime script mode, use `terminate_realtime();`, and to query realtime script mode use `is_realtime_script();`

`set_close_window_on_exit();` may be used to automatically close the client script window on terminating a realtime script

For an example script, look here https://pastebin.com/y5wrBnj8

### Autocompletes

You may specify a scripts autocompletes like #autos(test:1, test2:"hello", test3:"bitconnect");. The parser isn't that fancy so you may end up with slightly incorrect behaviour if you do eg #autos(test:1, test2:"test1:");

Warning: You MUST terminate an #autos statement with a semicolon otherwise itll break

### DB

\#db.f({example:"query"}) -> returns a cursor, use .array() or .first() to get the results. Takes an optional second projection argument

\#db.u({doot:{$exists:true}}, {$set:{doot:"nope"}}) -> finds all documents which have a key doot, and sets their key doot to 'nope'

\#db.i({key:"something", key2:"somethingelse"}) -> inserts a new document

\#db.r({key:something}) -> deletes all documents that have a key:something

### Misc

\#D("hello_there") -> overwrites your return value with any #D strings, which are appended together with newlines, works even if the script breaks except for timeouts but will overwrite errors. Only works for caller

print("some_string") -> gets printed onto the terminal prior to any return values, works even if the script breaks except for timeouts, will not overwrite errors. Works for everyone

timeout_yield(); -> cooperatively terminates the script if past the execution timeout. Print and #D only give output if the script is cooperatively terminated, which includes calling any function whatsoever, including any script, *s_call, db functions, and misc functions. The only method by which to get non cooperative termination is to timeout in pure JS code after 6 seconds, instead of the usual 5s timeout cap

### Other (non scriptable)

\#up scriptname -> will upload a script to the server

\#dry scriptname -> will test run a script upload to the server but not commit the changes

\#private scriptname -> will make a script private

\#public scriptname -> will make a script public

\#remove scriptname -> will remove a script from the server

\# -> lists all local scripts for that user

\#dir -> opens the script directory (shared between users)

\#edit scriptname -> creates or opens a script for editing, defaults to es6

\#edit_es6 scriptname -> creates or opens a script in es6/typescript mode. Will convert an es5 file to es6

\#edit_es5 scriptname -> same as above, but will convert an es6 file to es5

\#open scriptname -> opens a file for editing, but does not create

\#clear_autos -> clears autocompletes

\#shutdown -> shuts down the client

\#cls -> clears the terminal and chat

\#clear_term -> clears the main terminal window

\#clear_chat -> clears the main chat window

user \<username\> -> changes user, create automatically (will be changed in a future update)

\#delete_user \<username\> -> deletes a user, does not confirm or warn

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