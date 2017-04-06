# godot-xxENet-extension
Instructions how to modify [Godot Engine](https://godotengine.org/) v2.2 and v3.0 source code to add two additional functions to NetworkedMultiplayerENet and (optionally) SceneTree. 
**Please mind that I (rud01f) am not an experienced C++ programmer, and these modifications may (but not neccessary will) cause Godot Engine to be unstable. Please use it at your own risk, and if you are really interested in making this modifications stable, I suggest advice of experienced C++ programmer.** I will also be glad if you post an opinion about it or point out errors in code.    

These two functions, for SceneTree are:
`int disconnect_network_peer(int peer_id)` - disconnects client from server by its peer_id, returns number of peers disconnected; peer_id may be:
* greater than 1 - disconnects given peer_id from server (called by server instance)
* 1 - disconnects local (this) client from server (called by client instance)
* 0 - disconnects every remote peer (may be called from both client or server)
* negative - disconnects all peers except the (absolute value) of given (may be called from both client and server)
In all cases both sites will receive proper signals, so there should be no concern in handling disconnections - server will behave like client was disconnected ("network_peer_disconnected" signal), and client will be informed about disconnection from server ("server_disconnected" signal). 
`Dictionary get_network_peer_address(int peer_id)` - retrieves address of remote client (also works for server). peer_id should be greater than 1. This also works on client instance, providing 1 as peer_id returns server addres.
Returns Dictionary with keys "ip" (String) and "port" (integer) - for example {"ip": "192.168.33.11", "port": 7755}, if peer_id is invalid (negative or not existing), returned Dictionary will be empty ( result.empty() == true ). 
