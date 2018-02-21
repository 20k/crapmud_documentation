# crapmud_documentation

## API

### Client -> Server

Every client command should be preceded by "client_command ". A correct format for a request is "client_command #fs.scripts.trust()"

Calls which directly hook into the realtime chat system should be of the form "client_chat #hs.chats.send({channel:"<YOURCHAN>", msg:"<YOURMSG>"})"

The "client_chat " prefix simply suppresses script output from the server, so other commands could be piped through here if desired

The #up command follows the format client_command "#up scriptname <SCRIPT_DATA>"

Polling is performed by the request "client_poll" - this is the main driver that fetches chat so polls should be around ~1s in interval. This will probably be changed in the future because ddos'sing my own server is a poor move

The "client_command register client" command should be sent if no key.key file is present

### Server -> client

Responses from the server to "client_command "s are of the form "command <RESPONSE>"

Responses to "client_poll"s from the client are of the form "chat_api channel <DATA>". Data is formatted text with colour codes and should be displayed vanilla

The server sends no response for a "client_chat " command