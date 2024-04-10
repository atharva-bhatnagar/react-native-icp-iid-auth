# react-native-icp-iid-auth

A helper package for react native developers to authenticate using internet identity

## Installation

```bash
npm i --force react-native-icp-iid-auth
```

## Peer dependency [polyfills](https://www.npmjs.com/search?q=react-native-webview-crypto)

Required for usage of crypto in react native environment

```bash
npm i --force react-native-webview-crypto
```

```javascript
# use it in root (index.js/ts)

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

Required argument -> 

1. testing:Boolean, pass true if using in local environment.
2. idlfactories of type array of all the canisters ( Can be found from src/declarations)
3. canister ids array (array of strings), make sure they are respective to idlfactories array.

Example usage  ->

```javascript
 
import { StyleSheet, Text, TouchableOpacity, View } from 'react-native'
import React, { useContext, useEffect } from 'react'
import SplashScreen from 'react-native-splash-screen'
import { useRoute } from '@react-navigation/native'
import { UserContext } from '../index'
import { autoLogin } from './utils'
import { idlFactory } from './declarations/backend'

const App = () => {

  const route=useRoute()
  const {handleLogin}=route.params
  const {user,setUser} =useContext(UserContext)
  useEffect(()=>{
    SplashScreen.hide()
  },[])

  const login=async()=>{
    let idlFactories=[idlFactory]
    let canisterIDs=["bd3sg-teaaa-aaaaa-qaaba-cai"]
    let response=await handleLogin(true,idlFactories,canisterIDs)
    console.log(response)
    setUser(response.principle)
  }


  return (
    <View style={styles.app}>
      <Text style={styles.text}>React native login Test</Text>
      <Text style={styles.normalText}>Principal : {"\n\n"+user}</Text>
      <TouchableOpacity style={styles.btn} onPress={login}>
        <Text style={styles.btnText}>Login</Text>
      </TouchableOpacity>
      <TouchableOpacity style={[styles.btn,{marginTop:20}]} onPress={async()=>{autoLogin(setUser)}}>
        <Text style={styles.btnText}>Auto Login</Text>
      </TouchableOpacity>
    </View>
  )
}

```

## Considerations

1. The first element canister of idlfactories must contain a function called 'whoami' which returns the principle of the caller and is necessary for the confirmation of authentication through Internet Identity.
2. Make sure to use handleLogin method inside the root component itself (index.js) and then pass it to other components. If executed in other components like in the above example actors might generate some bugs if you will try to call these actors apart from the component where handleLogin is being executed and its child components.
3. This package is still under development and is not up to the mark for production level projects yet.


## License

[MIT](https://choosealicense.com/licenses/mit/)