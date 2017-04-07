# godot-xxENet-extension
Instructions how to modify [Godot Engine](https://godotengine.org/) v2.2 and v3.0 source code to add two additional functions to NetworkedMultiplayerENet and (optionally) SceneTree. 

**Please mind that I (rud01f) am not an experienced C++ programmer, and these modifications may (but not neccessary will) cause Godot Engine to be unstable. Please use it at your own risk, and if you are really interested in making this modifications stable, I suggest advice of experienced C++ programmer.** I will also be glad if you post an opinion about it or point out errors in code.    

**Modifications for Godot Engine 3.0 are not tested, and purely based on similarity of code to Godot 2.2**

## Detailed functions description:

### for SceneTree

`int disconnect_network_peer(int peer_id)` - disconnects client from server by its peer_id, returns number of peers disconnected; peer_id may be:
* greater than 1 - disconnects given peer_id from server (called by server instance)
* 1 - disconnects local (this) client from server (called by client instance)
* 0 - disconnects every remote peer (may be called from both client or server)
* negative - disconnects all peers except the (absolute value) of given (may be called from both client and server)
In all cases both sites will receive proper signals, so there should be no concern in handling disconnections - server will behave like client was disconnected ("network_peer_disconnected" signal), and client will be informed about disconnection from server ("server_disconnected" signal). 

`Dictionary get_network_peer_address(int peer_id)` - retrieves address of remote client (also works for server). peer_id should be greater than 1. This also works on client instance, providing 1 as peer_id returns server addres.
Returns Dictionary with keys "ip" (String) and "port" (integer) - for example {"ip": "192.168.33.11", "port": 7755}, if peer_id is invalid (negative or not existing), returned Dictionary will be empty ( result.empty() == true ). 

### for NetworkedMultiplayerENet

`int disconnect_peer(int peer_id)` - identical to `SceneTree.disconnect_network_peer(...)`

`Dictionary get_peer_address(int peer_id)` - identical to `SceneTree.get_network_peer_address(...)`

# Instructions

* for Godot Engine 2.2, with modification of both SceneTree and NetworkedMultiplayerENet: [STree_ENet_2_2.rst](STree_ENet_2_2.rst)
* for Godot Engine 2.2, with modification of NetworkedMultiplayerENet only: [ENet_only_2_2.rst](ENet_only_2_2.rst)
* for Godot Engine 3.0, with modification of both SceneTree and NetworkedMultiplayerENet: [STree_ENet_3_0.rst](STree_ENet_3_0.rst)
* for Godot Engine 3.0, with modification of NetworkedMultiplayerENet only: [ENet_only_3_0.rst](ENet_only_3_0.rst)

## Note of how NetworkedMultiplayerENet-only modification works

With this modification, there's no additonal functions on `SceneTree`, but things will work with NetworkedMultiplayerENet - just store instance reference of said ..ENet and call its fucntions. Remember to add it to SceneTree `get_tree().set_network_peer()` so events and transmission will be polled correctly.

# Example code that uses these functions:

```
# server.gd

var blacklist = ["127.0.0.1", "192.168.10.44"]

_ready():
  get_tree().connect("network_peer_connected", self, "_client_connected")
  # ...
  
func _client_connected(id):
  var adr = get_tree().get_network_peer_address(id)
  assert(not adr.empty())
  if adr.ip in blacklist:
    rpc_id(id, "kicked", "you have been banned from this server")
    get_tree().disconnect_network_peer(id)

# server.gd
remote kicked(reason):
  show_info("Kicked from server: " + reason)
```

**Happy coding!**
