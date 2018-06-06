# net_code documentation

## API

The official client uses websockets on 6760, the server will respond back to you in whichever mode you pick (text/binary), however be aware that commands that respond in binary will not work in text mode

As the websocket mode is new, there may be bugs, please let me know if you find any

#### Deprecated

The HTTP endpoint has been removed

client_poll and client_scriptargs are both deprecated and now undocumented. Use the JSON versions instead

Warning: The server's auth response was changed recently to fix a security vulnerability, please check the new response format!

### Client -> Server

Every client command should be preceded by "client_command ". A correct format for a request is "client_command #fs.scripts.core()"

Calls which directly hook into the realtime chat system should be of the form "client_chat #hs.msg.send({channel:"\<YOURCHAN\>", msg:"\<YOURMSG\>"})"

The "client_chat " prefix simply suppresses script output from the server, so other commands could be piped through here if desired

The #up command follows the format client_command "#up scriptname \<SCRIPT_DATA\>". Additionally, you may use #up_es6 to request es6/typescript compilation, which is slower

Polling is performed by the request "client_poll_json" - this is the main driver that fetches chat so polls should be around ~1s in interval. This will probably be changed in the future because ddos'sing my own server is a poor move

The "client_command register client" command should be sent if no key.key file is present. The response is "command_auth secret <128bytekey>". This 128 byte key should be saved and used to auth, it is not retrievable from the server

The "client_command auth client <128bytekey>" command should be sent to auth the client after a new connection is made to the server, or on reconnect

In the event that your websocket lib dislikes binary, you can use register client_hex and auth client_hex to process hex instead. The format is little endian. If you ask for hex, the response will be "command_auth secret_hex <128bytekeyashex>"

The "client_terminate_scripts JSON" command can be sent to terminate a realtime script. The JSON format is {"id":id}. If the id is -1, it will terminate any realtime script

The "client_script_keystrokes JSON" command can be sent to send keystrokes. The JSON format is {"id":id, "input_keys":["a", "enter", "space"], "released_keys":["up"], "pressed_keys":["lctrl"]}. Key format is love2d. Input keys are typed keys such as you might expect from a text editor, and pressed and released keys are the keys that have been pressed and released

The "client_script_info JSON" command can be sent to send generic script information. The current JSON format is {"id":id, "width":width, "height":height}. Width and height are in character sized units

#### Autocompletes

You may request an autocomplete from the server with the format "client_scriptargs_json script.name"

There is currently no way to batch autocompletes together, however with websockets being the default, this isn't so much of an issue

### Server -> client

Responses from the server to "client_command "s are of the form "command \<RESPONSE\>", except for auth client who's response is "command_auth secret <128bytekey>". Be aware that server responses generally may include colour codes EG \`Dhello\`, that you will be required to parse yourself

The full list of commands that will provoke a valid response are user <username>, #up, #dry, #remove, #public, #private, register client, auth client, auth client_hex and a JS command (which is any text which is not one of the former)

Responses to client_poll_json are in the format "chat_api_json JSON". The JSON format is: {"channels":[channel_list], "data":[{"channel":channel, "text":raw_chat_string}], "tells":[{"user":user, "text":raw_chat_string}]}, where array items may repeat indefinitely

The server sends no response for a "client_chat " command if you use the websocket endpoint. Responses from the server should be stashed in a file somewhere, and reloaded next script run

Async updates are sent to the client in the format "command_realtime_json JSON". The JSON format is {"id":id, "msg":msg, "width":width, "height":height, "close":boolean}. The id is globally unique across every possible script invocation, and uniquely identifies one script run. Width and height currently will only be sent once on script invocation, with no msg parameter

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

\#ls.net.view({user:"name", n:5}) -> views the network connections associated with a user or npc, gives position as well. In array:1 mode, returns an array where each member is an object with the properties name, x, y, z, links, stabilities

\#ls.net.map({user:"name", n:7}) -> views the network connections associated with a user or npc in 2d, with a max depth of n

\#ns.net.access({user:"name"}) -> access an NPC's command interface. Takes {add_user:"name"} or {remove_user:"name"} or {view_users:true}. Costs 200 cash to execute a command on an npc you have not added yourself to, in which case you must pass {confirm:true} to confirm payment

\#fs.net.hack({user:"name", extra_args:"example"}) -> hacks a user at user:"name", passes args forwards

\#ns.net.switch({user:"name"}) -> switches your terminal input to be run through this npc instead. Currently only works for npcs, and your main user that you are running on (from the user <username>) command

\#ns.net.move({user:"name", target:"destination"}) -> relinks a user or npc to the destination target. Must have a valid path

\#ns.net.path({user:"name", target:"destination", min_stability:23}) -> returns the visible path from user to target which has an optional minimum link stability. The stability calculated is the most direct path, not the visible path. In array:1 mode, returns {path:array, total_stability:num, avg_stability:num}

\#ns.net.modify({user:"name", target:"destination", delta:5}) -> changes the path stability between user and target overall by delta, may be negative. Takes an optional type:"use" or type:"view" to choose what kind of path to increase stability along. If you pass create:true will instead create a new link with stability delta

\#gal.* -> these functions are unfinished. #gal.list may be used to retrieve the list of sectors

### Realtime Scripting

To set a script to be realtime, use

`set_is_realtime_script();`

The script then is set into realtime mode. There are 4 callbacks that you can return:

on_draw, on_update(dt), on_input(char), and on_resize(dim). Dim is an object with a width and height property, in character sized units

For every 16ms you get 2ms of processing. You can terminate a realtime script with control-c, or by closing the associated window

To exit realtime script mode, use `terminate_realtime();`, and to query realtime script mode use `is_realtime_script();`

`set_close_window_on_exit();` may be used to automatically close the client script window on terminating a realtime script

`set_start_window_size({width:50, height:25});` will set the starting width and height of the window in character sized units

`is_key_down(char)` may be used to test if a key is down. Uses the love2d naming convention https://love2d.org/wiki/KeyConstant but bear in mind that not all keycodes are implemented yet, noteably any keycodes that would require shift to be pressed, numpad keys, F keys, and caps/scroll/numlock. F keys and *lock keys will never be supported, numpad is a maybe, and shift+1 keys are on the todo list

For an example script, look here https://pastebin.com/E9FE09mP

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

If your script is lowsec, all higher seclevel functions are available, eg. If you call ls_call, ls_call, ms_call, hs_call, and fs_call are available, but your script must be nullsec for ns_call to be available (so you cannot eval ns_call without at least one ns_call statement being made available to the parser as a regular statement)

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