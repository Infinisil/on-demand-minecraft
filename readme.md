# on-demand-minecraft

Goal:
Implement a <s>socket-started</s> systemd service that imitates a running minecraft server. When a whitelisted player tries to join, create a digital ocean droplet, start the actual server on it, and relay the connection to it

The rough sketch to implement this is as follows:
- If the actual server is running, relay the TCP connection to the actual server
- Otherwise read the first couple packets, which should be a handshake by the client as described in https://wiki.vg/Protocol#Handshake
  - If it's a status message, the user is looking at the multiplayer menu, which wants to know the status of the server. Imitate a running server by responding with a message indicating that the actual server isn't started and users will have to attempt a join to make it start. If the server is in the process of starting, indicate that in the message.
  - If it's a login message, the user double-clicked the server. Let the user send a Login Start message, which comes with a username. Check if the username is whitelisted. If it is, start the actual server and respond with an error message that it's being started. Otherwise let the user know that they're not whitelisted. If the server is in the process of starting, indicate that.

Previous work is in https://github.com/Infinisil/system/tree/2e3e3bfe3dd6904d91e944dba8bb0663e3d6ea0b/config/modules/on-demand-minecraft

Implementation steps:
- [x] Get a simple Haskell-based socket service working, so that one can ping it with e.g. `nc` and it responds with a pong. https://hackage.haskell.org/package/socket-activation can be used for this
- [x] Implement part of the minecraft server protocol such that status and login messages can be understood and replied to
  - [x] Status messages
  - [x] Login messages
- [x] Implement dynamically deciding what to do based on whether the actual server is online
- [x] Implement the server starting part with the digital ocean API: https://hackage.haskell.org/package/DOH
- [ ] Snoop the relayed connection to figure out how many people are online and delete the server again if nobody is online for 5 minutes. Can also be used to log to IRC
