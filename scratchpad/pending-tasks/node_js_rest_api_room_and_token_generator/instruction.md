Daily allows developers to programmatically create WebRTC rooms and manage permissions using its REST API. For secure meetings, rooms are often set to private and require authorized meeting tokens to enter.

You need to write a Node.js script using the native `fetch` API that creates a private Daily room and generates a meeting token for the room owner. 

**Constraints:**
- The script MUST send a `POST` request to `https://api.daily.co/v1/rooms` to create a room with `privacy: 'private'` and `enable_knocking: true`.
- The script MUST send a `POST` request to `https://api.daily.co/v1/meeting-tokens` to generate a token with `is_owner: true`.
- Both the room and the token MUST be configured to expire exactly 1 hour (3600 seconds) from the current time.
- The script must output the resulting room URL and the JWT token to the console.