# passport-battlepro

Passport strategy for authentication with [Battlepro](http://battlepro.com) through the OAuth 2.0 API.

## Usage
`npm install passport-battlepro --save`

#### Configure Strategy
The Battlepro authentication strategy authenticates users via a Battlepro user account and OAuth 2.0 token(s). A Battlepro API client ID, secret and redirect URL must be supplied when using this strategy. The strategy also requires a `verify` callback, which receives the access token and an optional refresh token, as well as a `profile` which contains the authenticated Battlepro user's profile. The `verify` callback must also call `cb` providing a user to complete the authentication.

```javascript
var BattleproStrategy = require('passport-battlepro').Strategy;

passport.use(new BattleproStrategy({
    clientID: 'id',
    clientSecret: 'secret',
    callbackURL: 'callbackURL',
    scope: ['identify', 'email'],
},
function(accessToken, refreshToken, profile, cb) {
    User.findOrCreate({ battleproId: profile.id }, function(err, user) {
        return cb(err, user);
    });
}));
```

#### Authentication Requests
Use `passport.authenticate()`, and specify the `'battlepro'` strategy to authenticate requests.

For example, as a route middleware in an Express app:

```javascript
app.get('/auth/battlepro', passport.authenticate('battlepro'));
app.get('/auth/battlepro/callback', passport.authenticate('battlepro', {
    failureRedirect: '/'
}), function(req, res) {
    res.redirect('/secretstuff') // Successful auth
});
```

#### Refresh Token Usage
In some use cases where the profile may be fetched more than once or you want to keep the user authenticated, refresh tokens may wish to be used. A package such as `passport-oauth2-refresh` can assist in doing this.

Example:

`npm install passport-oauth2-refresh --save`

```javascript
var BattleproStrategy = require('passport-battlepro').Strategy
  , refresh = require('passport-oauth2-refresh');

var battleproStrat = new BattleproStrategy({
    clientID: 'id',
    clientSecret: 'secret',
    callbackURL: 'callbackURL'
},
function(accessToken, refreshToken, profile, cb) {
    profile.refreshToken = refreshToken; // store this for later refreshes
    User.findOrCreate({ battleproId: profile.id }, function(err, user) {
        if (err)
            return done(err);

        return cb(err, user);
    });
});

passport.use(battleproStrat);
refresh.use(battleproStrat);
```

... then if we require refreshing when fetching an update or something ...

```javascript
refresh.requestNewAccessToken('battlepro', profile.refreshToken, function(err, accessToken, refreshToken) {
    if (err)
        throw; // boys, we have an error here.
    
    profile.accessToken = accessToken; // store this new one for our new requests!
});
```


## Examples
An Express server example can be found in the `/example` directory. Be sure to `npm install` in that directory to get the dependencies.

## License
Licensed under the ISC license. The full license text can be found in the root of the project repository.
