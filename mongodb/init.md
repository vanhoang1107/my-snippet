# Create User
```js
db.createUser({
    user: "PROJECT_live_play_svc",
    pwd:  passwordPrompt(),
    roles: [{ role: "readWrite", db: "PROJECT_play" } ]
})
```