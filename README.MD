# react-native-icp-iid-auth

A helper package for react native developers to authenticate using internet identity

## Installation

```bash
npm i --force react-native-icp-auth
```

## Peer dependency [polyfills](https://www.npmjs.com/search?q=react-native-webview-crypto)

Required for usage of crypto in react native environment

```bash
npm i --force react-native-webview-crypto
```

```javascript
// use it in root (index.js/ts)

import PolyfillCrypto from 'react-native-webview-crypto'

const RootComponent: React.FC = () => {

  return (
    <>
      <PolyfillCrypto />
      <NavigationContainer linking={linking}>
        <Stack.Navigator initialRouteName='Launch'>
          <Stack.Screen options={{headerShown:false}} name='Launch' component={App} initialParams={{handleLogin}}/>
        </Stack.Navigator>
      </NavigationContainer>
    </>
  );
};

```

## Usage

### Functions Provided  

**1. handleLogin :** Used for users to authenticate using Internet identity , this methods take 3 arguments : **environment** , **canisters** , **appName**(in lowercase).If any error occurs, it returns the error, if the authentication is complete then the Object that is returned by the function looks like :  
```javascript
  //object returned on successful authentication :

   {
      principal:"contains principal of the user",
      actors:[array of actors bound with IID]
   }

```
**Arguments used by handle login**   
  
**Environment :**  
  
```javascript
const environment={
    // if isTesting is true environment will be set to local , otherwise mainnet IID provider will be used, should be true only if dealing with the mainnet canisters
    isTesting:true,

    //these 3 are only applicable if isTesting is true, i.e. these are optional and should be used only when using the package to interact with local canisters

    middlepageID:"a3shf-5eaaa-aaaaa-qaafa-cai",//canister id of the locally deployed middlepage

    backendID:"a4tbr-q4aaa-aaaaa-qaafq-cai",//canister id of locally deployed backend containing whoami function

    backendIDL:idlFactory//idlfactory of locally deployed backend canister
  }

  //for mainnet canisters use environment as :
  const environment={
    isTesting:false
  }
```

**Canisters :**

  ```javascript
// array containing info about all the canisters developer needs to bind with the IID after authentication.

//idl factories can be imported from in src/declarations/<canister-name>/<canister-name>.did.js
import {idlFactory} from '../../src/declarations/backend/backend.did.js'

const canisters=[
    {
      idlFactory:idlFactory,

      //You can get canister IDs when you deploy you canisters using 'dfx deploy' or 'dfx deploy --network ic' 
      id:"a4tbr-q4aaa-aaaaa-qaafq-cai"
    },

  ]
```
**App Name :** Can be passed as a string, necessary for redirecting back to the app after successful authentication by the middle page.Also you need to insert this code in your **android / app / src / main / AndroidManifest.xml** :
```xml
<intent-filter>
  <action android:name="android.intent.action.VIEW" />
  <category android:name="android.intent.category.DEFAULT" />
  <category android:name="android.intent.category.BROWSABLE" />
  <data android:scheme="your-app-name" android:host="auth" />
</intent-filter>

```

**2. autoLogin :** Should be placed inside the useeffect hook of the initialRoute. This function scans if any previous IID authenticated data is stored locally and if any previous data is found, it automatically authenticates the user. for takes 2 arguments **environment** and **canisters** which are same as described for the handleLogin function.Consoles err if something went wrong, Return type of this function resembles this object :
```javascript
// Response If previous data is found : 
{
 found:true,
 principal:"user's principal",
 actors:[array of IID bound actors]
}

// Response If no data found : 
{
 found:false,
 msg:"NO previous user data found"
}
```

**3. handleLogout :** Takes no parameters and returns nothing. Clears the data stored locally on mobile related to IID authentication and reload the app in order to prevent any agent state clashed resulting in unnecessary errrors and warnings. 

### Example usage for local canisters :


Code for backend and middlePage canisters can be found in [this repository](https://github.com/atharva-bhatnagar/react-native-package-test).
take the two actors and deploy them locally, use dfx generate command to get the declarations for the same

```javascript
  import {idlFactory} from '../../src/declarations/canister1/canister1.did.js'
  import {idlFactory as idlFactory2} from '../../src/declarations/canister2/canister2.did.js'

  const environment={
    isTesting:true,
    middlepageID:"a3shf-5eaaa-aaaaa-qaafa-cai", //canister id of the locally deployed middlepage
    backendID:"a4tbr-q4aaa-aaaaa-qaafq-cai", //canister id of locally deployed backend containing whoami function
    backendIDL:idlFactory //idlfactory of locally deployed backed canister
  }
  const appName="demo"
  const canisters=[
    {
      idlFactory:idlFactory,
      id:"a4tbr-q4aaa-aaaaa-qaafq-cai"
    },
    {
      idlFactory:idlFactory2,
      id:"b3tbr-q4aaa-aaaaa-qaafq-cai"
    },
  ]
  //for logging in, returns principal and bound actors
  let res=await handleLogin(environment,canisters,appName)
  
  //for automatic logging in, returns principal and bound actors
  let res2=await autologin(environment,canisters)

  //for user logout 
  logout()
```

### Example usage for mainnet canisters :

```javascript
  import {idlFactory} from '../../src/declarations/canister1/canister1.did.js'
  import {idlFactory as idlFactory2} from '../../src/declarations/canister2/canister2.did.js'

  const environment={
    isTesting:false,
  }
  const appName="demo"
  const canisters=[
    {
      idlFactory:idlFactory,
      id:"a4tbr-q4aaa-aaaaa-qaafq-cai"
    },
    {
      idlFactory:idlFactory2,
      id:"b3tbr-q4aaa-aaaaa-qaafq-cai"
    },
  ]
  //for logging in, returns principal and bound actors
  let res=await handleLogin(environment,canisters,appName)
  
  //for automatic logging in, returns principal and bound actors
  let res2=await autologin(environment,canisters)

  //for user logout 
  logout()
```

In-depth Example usage of this package can be found in [this repository](https://github.com/atharva-bhatnagar/react-native-package-test).



## Considerations

1. Make sure to use handleLogin method inside the root component itself (index.js) and then pass it to other components using a context provider to avoid any errors.
2. For local testing of the project you need to setup [local internet identity provider](https://internetcomputer.org/docs/current/developer-docs/web-apps/user-login/internet-identity/integrate-internet-identity). Also deploy your own middlepage and backend canister necessary for authentication and pass their IDs and backend idlfactory inside environment argument when using **handleLogin** and **autoLogin** functions. The code for the middlepage frontend canisters,backend canister as well as declarations( idlfactory ) can be found in [this repository](https://github.com/atharva-bhatnagar/react-native-package-test), You can navigate through the demo project and find the canister code with the help of dfx.json.


## License

[MIT](https://choosealicense.com/licenses/mit/)