Extension of Godot Engine 2.2 xxENet multiplayer support.
=========================================================

**Remainder: be aware that provided modification is written by unexperienced C++ programmer, therefore applying them may make Godot Engine unstable or lead to memory leaks. Please consult with experienced C++ programmer, preferably someone with basic knowledge about Godot Engine.**

Below is a list of file paths and instructions - "groot" (no pun intended) refers to Godot Engine source code root directory.
These modification will add functions to NetworkedMultiplayerENet only.

* ``groot/modules/enet/networked_multiplayer_enet.h``

::

  // after "public:" keyword of NetworkedMultiplayerENet class:

  virtual int disconnect_peer(int peer_id);
  virtual Dictionary get_peer_address(int peer_id);

* ``groot/modules/enet/networked_multiplayer_enet.cpp``

::

  // inside "NetworkedMultiplayerPeer::_bind_methods()" method:
  
  ObjectTypeDB::bind_method(_MD("disconnect_peer:int", "peer_id"), &NetworkedMultiplayerENet::disconnect_peer);
  ObjectTypeDB::bind_method(_MD("get_peer_address:Dictionary", "peer_id"), &NetworkedMultiplayerENet::get_peer_address);
  
  // in the main scope (outside any method/function):
  
  int NetworkedMultiplayerENet::disconnect_peer(int peer_id) {
    int cnt = 0;
    const enet_uint32 DATA_ = 0;
    if (peer_id == 0) {
      for (Map<int, ENetPeer*>::Element *F=peer_map.front();F;F=F->next()) {
        ENetPeer *peer_ = F->get();
        enet_peer_disconnect(peer_, DATA_);
        cnt++;
      }
      return cnt;
    } else if (peer_id < 0) {
      for (Map<int, ENetPeer*>::Element *F=peer_map.front();F;F=F->next()) {
        int pid = F->key();
        ENetPeer *peer_ = F->get();
        if (pid != ABS(peer_id)) {
          enet_peer_disconnect(peer_, DATA_);
          cnt++;
        }
      }
      return cnt;
    } else if (peer_id > 0) {
      Map<int, ENetPeer*>::Element *E = NULL;
      E = peer_map.find(ABS(peer_id));
      if (E) {
        enet_peer_disconnect(E->get(), DATA_);
        cnt = 1;	
      }
      return cnt;
    }
    return 0;
  }

  Dictionary NetworkedMultiplayerENet::get_peer_address(int peer_id) {
    Dictionary ret = NULL;
    Map<int, ENetPeer*>::Element *E = NULL;
    E = peer_map.find(ABS(peer_id));
    if (E) {
      ENetAddress addr = E->get()->address;
      int host = addr.host;
      String value = itos(host & 0xFF) + "." + itos((host >> 8) & 0xFF) + "." + itos((host >> 16) & 0xFF) + "." + itos((host >> 24) & 0xFF);
      ret["ip"] = value;
      ret["port"] = addr.port;
    }
    return ret;
  }

** end of modification**
