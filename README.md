# eloquafy

## How to install

$> npm install eloquafy

## How to use

Eloqua is base on promise and psharky, and psharky is base on Nodejs Events.

When you have developed an app and are ready to start testing it, you need to register it with Eloqua, Once registered you need to define Application lifecycles endpoints

### Enable application

The enable lifecycle events allows you to do client authorization flow you need to aquire atleast 3 field and pass it here.
these api returns the authorization endpoint inside an array

```
let emitter = require('psharky');

// these is where the library gets added
require('eloquafy')(emitter);


let redirect_uri = config.serverurl + '<callback>';
let enable_uri_option = {
  install_id: req.query.InstallId ,
  redirect_uri: redirect_uri,
  client_id: req.query.AppId
};
let enable_app = emitter.invokeHook('eloqua::application::enable',enable_uri_option);
enable_app.then(function(response){
 // record the CallbackUrl taken from query parameters from here, WIthout the callback URL you will not be able to complete the installation flow if your app requires configuring (for example, to use OAuth).
 // catch the response here which is an authorization url and redirect 
 res.redirect(response[0]);  
},function(error){
    
});

```

install_id - or in eloqua InstallId, is a templated parameter used in AppCloud service apps. Its value is the GUID for the user's installation of the AppCloud App. Whenever a user installs an app, a new InstallId is created.

redirect_uri - is where your application going to complete the installation process after client authorization.

client_id - or in eloqua AppId, is a templated parameter used in AppCloud service apps. Its value is the GUID-based ID of the app making the service call. Each AppCloud app has a single unique AppId.


### Enable application oauth callback

The enable callback is the 2nd step from above and it will return the required tokens and base url of the consumer
thse api returns the tokens inside an array
```

let redirect_uri = config.serverurl + '<callback>';
  let callback_option = {
    client_id: config.client_id ,
    client_secret: config.client_secret,
    authorization_code: req.query.code,
    redirect_uri: redirect_uri
  };
  let enable_callback = emitter.invokeHook('eloqua::application::enable::callback',callback_option);
  enable_callback.then(function(response){
    let token = enable_uri_res[0];

    // token.refresh_token,
    // token.access_token,
    // token.eloqua_base_url

    // you need to redirect back to CallbackUrl from the first step to confirm that installation is successful
    res.redirect(consumer.CallbackUrl);
  }, function(error){


  });

```

client_id and client_secret are from your eloqua registered application details
authorization is from the query parameters provided by eloqua
redirect_uri is the endpoint where this code is placed

### Verify Eloqua Request

To verify request from eloqua we submit originalUrl, method, client_id and client_secret, we will receive an array with object status inside.

```

      let verify_options = {
        "originalUrl": config.serverurl + req.originalUrl,           	                        
        "method" : req.method,
        "client_id" : config.client_id,
        "client_secret" : config.client_secret
      };
      let verify = emitter.invokeHook('eloqua::request::verify',verify_options);
      verify.then(function(verify_res){
        if(verify_res[0].status){
          next();
        }
        else{
          res.status(400).json("Oauth Verification failed");
        }
      },function(err){
        res.status(400).json(err);
      });

```