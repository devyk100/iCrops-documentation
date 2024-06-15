# The iCrops app

### The app is made using `react native cli`, and can be built for iOS, Android, and Web.

#### It uses `Typescript` as it's programming language. And mainly react libraries like redux, axios, and other react-native-community libraries for other functionalities.

# Project Structure

```
- android
- app
 - assets // contains all the images used in the app
 - auth
    Login.tsx
    MainAuth.tsx
   - Signup.tsx
 - components
   - datacollection
      CCE.tsx
      CropInformation.tsx
      Description.tsx
      Location.tsx
      LocationOffset.tsx
      Main.tsx
      QualityControl.tsx
      SeasonSelection.tsx
    Button.tsx
    CustomDatePicker.tsx
    CustomModal.tsx
    CustomModal2.tsx
    CustomModalWithTextBox.tsx
    FormCameraHandle.tsx
    LandingPage.tsx
    MapChooseLocation.tsx
    SmallDraw.tsx
 - data
    index.ts
 - features
    DataCollectionSlice.tsx
    LocationSlice.tsx
    UISlice.tsx
 - help
    HelpPage.tsx
 - localStorage
    index.ts
 - location
    getLocation.tsx
 - maps
    MapScreen.tsx
 - networking
    index.ts
 - store
    index.ts
 - types
    index.tsx
- ios
- node_modules
app.json
App.tsx
babel.config.js
Gemfile
index.js
jest.config.js
package.json
README.md
tsconfig.json
yarn.lock
```

There's no need to touch the `android` and `ios` folders, unless you've added a dependency that requires some native code changes like `Storage Permissions`,other permissions, etc.,

### For sharing the app, never share the `node_modules`, and `android/build` folder, delete them before sharing as they can slow down your indexing while sharing, and any user can automatically build them again.

## To run the app for DEBUGGING and development scenarios on `android`

- `node.js` must be installed and available on the path., i.e command line commands like `npm` and `node` should work in `bash`, `powershell`, `cmd`, or `iterm`.
- `yarn` package manager is recommended for this project, install it using `npm install --global yarn`, as this is much faster and efficient than npm, or if you're having errors, you can proceed with just `npm`
- you must have android studio installed and must be available on the path `ANDROID_HOME`
- jdk 17 or above must be installed and available on the path `JAVA_HOME`
- do a `yarn add` or `npm install` to install all the packages, or if a new dependency was added
- run `yarn start` or `npx react-native start`, this should start a server preferrably at port `8081`, and you'll see all your logs of the app in this terminal
- make sure you have an emulator opened, or a device connected with `USB Debugging` is enabled.
- run `yarn run android` or `npx react-native run-android`, this should install the app on the device.
- Now you can proceed with your development and should be able to see a `fast refresh` the moment you change any code.

## To make the production build `apk` for `android`

- run `cd android && gradlew assembleRelease`
- this will generate a `apk` file in `android/app/build/outputs/release/` and then you can use it for production.

# Understanding the code.

The entry point of this app like any other node.js project is the root `index.js` file, reading it from there should give you an idea how the flow of the project goes.

- index.js

```
/**
 * @format
 */
import 'react-native-gesture-handler';


import {AppRegistry} from 'react-native';
import App from './App';
import {name as appName} from './app.json';

AppRegistry.registerComponent(appName, () => App);

```

- App.tsx. The code for navigation through drawers at the main page after login, and also the login logic by fetching and checking the local storage for the jwts.

```
import * as React from 'react';
import {Button, Image, Text, View} from 'react-native';
import {
  DrawerContentScrollView,
  DrawerItem,
  DrawerItemList,
  createDrawerNavigator,
} from '@react-navigation/drawer';
import {NavigationContainer, NavigationState} from '@react-navigation/native';
import LandingPage from './app/components/LandingPage';
import DataCollection from './app/components/datacollection/Main';
import {Provider, useSelector} from 'react-redux';
import store from './app/store';
import BootSplash from 'react-native-bootsplash';
import {
  selectDegreesToNorth,
  selectLocation,
} from './app/features/LocationSlice';
// @ts-ignore
import compassImage from './app/assets/compass.png';
import Login from './app/auth/Login';
import MainAuth from './app/auth/MainAuth';
import {useMMKVString} from 'react-native-mmkv';
import MapScreen from './app/maps/MapScreen';

function NotificationsScreen({navigation}: {navigation: any}) {
  return (
    <View style={{flex: 1, alignItems: 'center', justifyContent: 'center'}}>
      <Button onPress={() => navigation.goBack()} title="Go back home" />
    </View>
  );
}

function DataCollectionHeader() {
  const locationData = useSelector(selectLocation);
  const degreesToNorth = useSelector(selectDegreesToNorth);
  return (
    <>
      {/* <View style={{
      height:"20%",
      width:"100%",
      backgroundColor:"red"
    }}> */}
      <View
        style={{
          flexDirection: 'row',
          justifyContent: 'space-between',
          width: '100%',
          alignItems: 'center',
        }}>
        <Text
          // @ts-ignore
          style={{
            color: 'red',
            fontSize: 20,
            fontWeight: 600,
          }}>
          Data Collection
        </Text>
        <View
          style={{
            flexDirection: 'row',
            justifyContent: 'center',
            alignItems: 'center',
          }}>
          <Image
            source={compassImage}
            style={{
              height: 40,
              width: 40,
              transform: [{rotate: `${-(40 + degreesToNorth)}deg`}],
            }}></Image>
          <Text
            style={{
              color: 'blue',
            }}>
            {locationData.accuracy
              ? Math.round(locationData.accuracy) + ' m'
              : null}
          </Text>
        </View>
      </View>
      {/* </View> */}
    </>
  );
}

const Drawer = createDrawerNavigator();

function CustomDrawerContent(props: any) {
  const [email, setEmail] = useMMKVString('user.email');
  const [jwt, setJwt] = useMMKVString('user.jwt');
  return (
    <DrawerContentScrollView {...props}>
      <DrawerItem
        label={'Hi ' + email}
        onPress={() => console.log('does something')}
        // @ts-ignore
        labelStyle={{
          fontSize: 18,
          fontWeight: 300,
        }}
      />
      <DrawerItemList {...props} />
      <DrawerItem
        label={'Logout'}
        onPress={() => {
          setEmail(undefined);
          setJwt(undefined);
        }}
        labelStyle={{}}
      />
    </DrawerContentScrollView>
  );
}

export default function App() {
  const [email, setEmail] = useMMKVString('user.email');
  const [jwt, setJwt] = useMMKVString('user.jwt');
  if (jwt == undefined) return <MainAuth />;
  else
    return (
      <Provider store={store}>
        <NavigationContainer
          onReady={() => {
            console.log('Loaded the main screen');
            BootSplash.hide({fade: true});
          }}>
          <Drawer.Navigator
            initialRouteName="ICRISAT iCrops"
            screenOptions={{
              headerStyle: {backgroundColor: '#90EE90', borderColor: 'red'},
              drawerContentStyle: {
                backgroundColor: 'white',
              },
            }}
            drawerContent={props => <CustomDrawerContent {...props} />}>
            <Drawer.Screen
              name="ICRISAT iCrops"
              options={{
                drawerStatusBarAnimation: 'slide',
                // overlayColor: "yellow"
              }}
              component={LandingPage}
            />
            <Drawer.Screen
              name="datacollection"
              options={{
                headerTitle: () => <DataCollectionHeader />,
                title: 'Data Collection',
              }}
              component={DataCollection}
            />
            <Drawer.Screen
              name="mapplotting"
              options={{
                title: 'Map Plotting',
              }}
              component={MapScreen}
            />
          </Drawer.Navigator>
        </NavigationContainer>
        {/* <ImageMarker></ImageMarker> */}
        {/* <MapChooseLocation></MapChooseLocation> */}
      </Provider>
    );
}

```

- app/auth/Login.tsx

```
import axios from 'axios';
import {useCallback, useRef, useState} from 'react';
import {
  Alert,
  Button,
  ScrollView,
  Text,
  TextInput,
  TextInputComponent,
  TextInputProps,
  View,
} from 'react-native';
import {BACKEND_URL} from '../networking';
import {setJwtEmail} from '../localStorage';
// import "dotenv/config"

export default function () {
  const backendUrl = BACKEND_URL;
  const [email, setEmail] = useState<string>('');
  const [password, setPassword] = useState<string>('');
  const submit = useCallback(async () => {
    // console.log(email, "is the email", password, "is the password here", backendUrl)
    // Alert.alert('going inside the BACKEND URL', BACKEND_URL);
    const response = await axios.post(BACKEND_URL + 'api/v1/user/login/', {
      email,
      password,
    });
    // Alert.alert('going inside the BACKEND URL', response.data);
    console.log(response);
    if (response.data.success) {
      // console.log("GOT THE RESPONSE", response.data)
      setJwtEmail(response.data.jwt, response.data.email);
    } else {
      Alert.alert('Login failed, check the password and email id');
    }
  }, [email, password]);
  return (
    <>
      <ScrollView
        style={{
          flexDirection: 'column',
          // alignItems: "center",
          // justifyContent: "center"
        }}>
        <View
          style={{
            flex: 1,
            flexDirection: 'column',
            alignItems: 'center',
            justifyContent: 'center',
          }}>
          <Text
            style={{
              fontSize: 24,
              padding: 20,
              color: 'black',
              fontWeight: '600',
            }}>
            Login to iCrops App
          </Text>
        </View>
        <View
          style={{
            flex: 1,
            flexDirection: 'column',
            alignItems: 'center',
            justifyContent: 'center',
          }}>
          <Text
            style={{
              color: 'black',
              fontSize: 20,
              flex: 1,
              textAlign: 'left',
              width: '80%',
            }}>
            Email ID
          </Text>
          <TextInput
            style={{
              backgroundColor: 'white',
              // borderBlockColor: 'grey',
              borderWidth: 1,
              width: '80%',
              borderRadius: 10,
              color: 'black',
              fontSize: 18,
              paddingLeft: 8,
            }}
            value={email}
            onChangeText={setEmail}></TextInput>
        </View>
        <View
          style={{
            flex: 1,
            flexDirection: 'column',
            alignItems: 'center',
            justifyContent: 'center',
          }}>
          <Text
            style={{
              color: 'black',
              fontSize: 20,
              flex: 1,
              textAlign: 'left',
              width: '80%',
            }}>
            Password
          </Text>
          <TextInput
            style={{
              backgroundColor: 'white',
              // borderBlockColor: 'grey',
              borderWidth: 1,
              width: '80%',
              borderRadius: 10,
              fontSize: 18,
              color: 'black',
              paddingLeft: 8,
            }}
            onChangeText={setPassword}
            value={password}
            secureTextEntry={true}></TextInput>
        </View>
        <View
          style={{
            flex: 1,
            flexDirection: 'column',
            alignItems: 'center',
            justifyContent: 'center',
            marginTop: 20,
          }}>
          <View
            style={{
              width: '80%',
            }}>
            <Button onPress={() => submit()} title="Login"></Button>
          </View>
        </View>
      </ScrollView>
    </>
  );
}

```

- app/auth/MainAuth.tsx . It has the code for the bottom tabs navigation and integrating together Login.tsx and SignUp.tsx

```
import {createBottomTabNavigator} from '@react-navigation/bottom-tabs';
import Login from './Login';
import Signup from './Signup';
import {NavigationContainer, NavigationState} from '@react-navigation/native';
import {Image} from 'react-native';
//@ts-ignore
import loginIcon from '../assets/login.png';
//@ts-ignore
import signupIcon from '../assets/signup.png';
import BootSplash from 'react-native-bootsplash';
const Tab = createBottomTabNavigator();

export default function MainAuth() {
  return (
    <NavigationContainer
      onReady={() => {
        BootSplash.hide({fade: true});
        console.log('Loaded the main screen');
      }}>
      <Tab.Navigator>
        <Tab.Screen
          name="Login"
          component={Login}
          options={{
            tabBarIcon: () => <Image source={loginIcon} />,
          }}
        />
        <Tab.Screen
          name="Sign Up"
          component={Signup}
          options={{
            tabBarIcon: () => <Image source={signupIcon} />,
          }}
        />
      </Tab.Navigator>
    </NavigationContainer>
  );
}
```

- The app/components folder has the following components, and their accomplished tasks are as same as their namees:
  - Button
  - CustomDatePicker
  - CustomModal
  - CustomModal2 - slightly different than the first one
  - CustomModalWithTextBox
  - FormCameraHandle
  - LandingPage
  - MapChooseLocation
  - SmallDrawer

Now we describe the code for each

Button.tsx

```
import { ReactNode } from "react";
import { TouchableOpacity, Text } from "react-native";
// import {  } from "react-native-reanimated/lib/typescript/Animated";

export default function({children, handler}: {
    children: ReactNode,
    handler?: () => void
}){
    return (
        <>
        <TouchableOpacity style={{
            backgroundColor:"#d4d4d4",
            flex:1,
            padding:10,
            margin:20,
            marginTop: 0
        }} onPress={handler}>
            <Text style={{
                color: "black",
                textAlign:"center"
            }}>{children}</Text>
        </TouchableOpacity>
        </>
    )
}
```

CustomDatePicker.tsx

```
import { useState } from "react";
import { Button, Text } from "react-native";
import DatePicker from "react-native-date-picker";

export default function({date, setDate}: {
    date: Date;
    setDate: (date: Date) => void;
}){
    const [open, setOpen] = useState(false);
    // const [date, setDate] = useState(new Date());
    return (
        <>
                <Text style={{
          backgroundColor:"#cefac0",
          borderRadius:14,
          color:"black",
          fontSize:18,
          marginHorizontal:5,
          padding:6
        }}>
          {`${date.getDate()}/${date.getMonth()+1}/${date.getFullYear()} `}
        </Text>
        <Button title='pick date' onPress={() => setOpen(true)}>
        </Button>
        <DatePicker
        modal
        open={open}
        date={date}
        mode="date"
        onConfirm={(date) => {
          console.log(date)
          setOpen(false)
          setDate(date)
        }}
        onCancel={() => {
          setOpen(false)
        }}
      />
        </>
    )
}
```

CustomModal.tsx

```
import {useEffect, useState} from 'react';
import {
  Alert,
  Button,
  Modal,
  Pressable,
  ScrollView,
  Text,
  TouchableOpacity,
  View,
} from 'react-native';
import {useSelector} from 'react-redux';
import {selectResetter} from '../features/LocationSlice';

type dataType = {
  value: number;
  title: string;
};

export default function ({
  data,
  action,
}: {
  data: dataType[];
  action: (payload: any) => any;
}) {
  const [modalVisible, setModalVisible] = useState(false);
  const [modalValue, setModalValue] = useState(1);
  const [firstTime, setFirstTime] = useState(true);
  const resetterValue = useSelector(selectResetter);
  useEffect(() => {
    setModalValue(1);
    console.log(resetterValue);
  }, [resetterValue]);
  return (
    <>
      {resetterValue ? null : null}
      <Modal
        animationType="slide"
        transparent={true}
        visible={modalVisible}
        onRequestClose={() => {
          //   Alert.alert('Modal has been closed.');
          setModalVisible(!modalVisible);
        }}>
        <View
          style={{
            backgroundColor: 'white',
            height: '100%',
            padding: 10,
            borderTopColor: 'red',
            borderTopWidth: 3,
            flexDirection: 'row',
          }}>
          <TouchableOpacity
            style={{
              padding: 10,
              flex: 1,
              backgroundColor: 'grey',
              height: 40,
              borderRadius: 40,
            }}
            onPress={() => setModalVisible(!modalVisible)}>
            <Text
              style={{
                color: 'black',
              }}>
              Close
            </Text>
          </TouchableOpacity>
          <ScrollView
            style={{
              width: '65%',
              margin: 10,
            }}>
            {data.map((value: any, id) => {
              return (
                <TouchableOpacity
                  key={id}
                  onPress={e => {
                    setModalValue(value.value);
                    setModalVisible(t => !t);
                    action(value.title);
                    setFirstTime(false);
                  }}
                  // key={value.title}
                  style={{
                    padding: 5,
                    margin: 5,
                    borderBottomWidth: 1,
                    borderBottomColor: 'grey',
                  }}>
                  {modalValue != value.value ? (
                    <Text style={{color: 'black', fontSize: 18}}>
                      {value.title}
                    </Text>
                  ) : (
                    <Text style={{color: 'blue', fontSize: 18}}>
                      {value.title}
                    </Text>
                  )}
                </TouchableOpacity>
              );
            })}
          </ScrollView>
        </View>
      </Modal>

      <Button
        title={firstTime ? 'Unselected' : data[modalValue - 1]?.title}
        onPress={() => setModalVisible(t => !t)}></Button>
    </>
  );
}

```

CustomModal2.tsx

```
import {useEffect, useState} from 'react';
import {
  Alert,
  Button,
  Modal,
  Pressable,
  ScrollView,
  Text,
  TouchableOpacity,
  View,
} from 'react-native';
import {useSelector} from 'react-redux';
import {selectResetter} from '../features/LocationSlice';

type dataType = {
  value: number;
  title: string;
};

export default function ({
  data,
  action,
}: {
  data: dataType[];
  action: (payload: any) => any;
}) {
  const [modalVisible, setModalVisible] = useState(false);
  const [modalValue, setModalValue] = useState(1);
  const [firstTime, setFirstTime] = useState(true);
  const resetterValue = useSelector(selectResetter);
  useEffect(() => {
    setModalValue(1);
    console.log(resetterValue);
  }, [resetterValue]);
  return (
    <>
      <Modal
        animationType="slide"
        transparent={true}
        visible={modalVisible}
        onRequestClose={() => {
          //   Alert.alert('Modal has been closed.');
          setModalVisible(!modalVisible);
        }}>
        <View
          style={{
            backgroundColor: 'white',
            height: '100%',
            padding: 10,
            borderTopColor: 'red',
            borderTopWidth: 3,
            flexDirection: 'row',
          }}>
          <TouchableOpacity
            style={{
              padding: 10,
              flex: 1,
              backgroundColor: 'grey',
              height: 40,
              borderRadius: 40,
            }}
            onPress={() => setModalVisible(!modalVisible)}>
            <Text
              style={{
                color: 'black',
              }}>
              Close
            </Text>
          </TouchableOpacity>
          <ScrollView
            style={{
              width: '65%',
              margin: 10,
            }}>
            {data.map((value: any) => {
              return (
                <TouchableOpacity
                  onPress={e => {
                    setModalValue(value.value);
                    setModalVisible(t => !t);
                    action(value.title);
                    setFirstTime(false);
                  }}
                  key={value.title}
                  style={{
                    padding: 5,
                    margin: 5,
                    borderBottomWidth: 1,
                    borderBottomColor: 'grey',
                  }}>
                  {modalValue != value.value ? (
                    <Text style={{color: 'black', fontSize: 18}}>
                      {value.title}
                    </Text>
                  ) : (
                    <Text style={{color: 'blue', fontSize: 18}}>
                      {value.title}
                    </Text>
                  )}
                </TouchableOpacity>
              );
            })}
          </ScrollView>
        </View>
      </Modal>
      {/* <TouchableOpacity
                style={{
                  backgroundColor: '#d4d4d4',
                  padding: 10,
                  marginTop: 0,
                }}
                onPress={() => setModalVisible(t => !t)}>
                <Text
                  style={{
                    color: 'black',
                    textAlign: 'center',
                  }}>
                  {data[modalValue-1]?.title}
                </Text>
              </TouchableOpacity> */}
      <View
        style={{
          // marginHorizontal: 3,
          flexDirection: 'column',
          alignItems: 'center',
          flex: 7,
        }}>
        <Button
          title={firstTime ? 'Unselected' : data[modalValue - 1]?.title}
          onPress={() => setModalVisible(t => !t)}></Button>
        <Text
          style={{
            color: 'black',
            fontSize: 12,
          }}>
          Select Season
        </Text>
      </View>
    </>
  );
}

```

CustomModalWithTextBox.tsx

```
import {useEffect, useState} from 'react';
import {
  Alert,
  Button,
  Modal,
  Pressable,
  ScrollView,
  Text,
  TextInput,
  TouchableOpacity,
  View,
} from 'react-native';
import {useSelector} from 'react-redux';
import {selectResetter} from '../features/LocationSlice';

type dataType = {
  value: number;
  title: string;
};

export default function ({
  data,
  action,
}: {
  data: dataType[];
  action: (payload: any) => any;
}) {
  const [modalVisible, setModalVisible] = useState(false);
  const [modalValue, setModalValue] = useState(1);
  const [newCropToBeAdded, setNewCropToBeAdded] = useState(false);
  const [newCropValue, setNewCropValue] = useState<string>('');
  const [firstTime, setFirstTime] = useState(true);
  const resetterValue = useSelector(selectResetter);
  useEffect(() => {
    setModalValue(1);
    console.log(resetterValue);
  }, [resetterValue]);
  return (
    <>
      <Modal
        animationType="slide"
        transparent={true}
        visible={modalVisible}
        onRequestClose={() => {
          //   Alert.alert('Modal has been closed.');
          setModalVisible(!modalVisible);
        }}>
        <View
          style={{
            backgroundColor: 'white',
            height: '100%',
            padding: 10,
            borderTopColor: 'red',
            borderTopWidth: 3,
            flexDirection: 'row',
          }}>
          <TouchableOpacity
            style={{
              padding: 10,
              flex: 1,
              backgroundColor: 'grey',
              height: 40,
              borderRadius: 40,
            }}
            onPress={() => setModalVisible(!modalVisible)}>
            <Text
              style={{
                color: 'black',
              }}>
              Close
            </Text>
          </TouchableOpacity>
          <ScrollView
            style={{
              width: '65%',
              margin: 10,
            }}>
            {newCropToBeAdded ? (
              <View
                style={{
                  flexDirection: 'row',
                  justifyContent: 'center',
                  alignItems: 'center',
                }}>
                <TextInput
                  style={{
                    padding: 5,
                    margin: 5,
                    fontSize: 18,
                    borderBottomWidth: 1,
                    borderBottomColor: 'green',
                    flex: 3,
                    color: 'blue',
                  }}
                  value={newCropValue}
                  onChangeText={setNewCropValue}></TextInput>
                <Button
                  title="done"
                  color={'grey'}
                  onPress={() => {
                    action(newCropValue);
                    setFirstTime(false);
                    setModalVisible(t => !t);
                  }}></Button>
              </View>
            ) : null}

            {!newCropToBeAdded ? (
              <TouchableOpacity
                onPress={e => {
                  setNewCropToBeAdded(true);
                }}
                style={{
                  backgroundColor: 'lightgreen',
                  alignItems: 'center',
                  borderRadius: 25,
                  padding: 10,
                  flexDirection: 'row',
                }}>
                <Text
                  style={{
                    color: 'black',
                    flex: 6,
                    textAlign: 'center',
                    fontSize: 18,
                  }}>
                  Add crop
                </Text>
                <Text
                  //@ts-ignore
                  style={{
                    fontSize: 20,
                    fontWeight: 1000,
                  }}>
                  +
                </Text>
              </TouchableOpacity>
            ) : null}
            {data.map((value: any) => {
              return (
                <TouchableOpacity
                  onPress={e => {
                    setModalValue(value.value);
                    setModalVisible(t => !t);
                    action(value.title);
                    setFirstTime(false);
                  }}
                  key={value.title}
                  style={{
                    padding: 5,
                    margin: 5,
                    borderBottomWidth: 1,
                    borderBottomColor: 'grey',
                  }}>
                  {modalValue != value.value ? (
                    <Text style={{color: 'black', fontSize: 18}}>
                      {value.title}
                    </Text>
                  ) : (
                    <Text style={{color: 'blue', fontSize: 18}}>
                      {value.title}
                    </Text>
                  )}
                </TouchableOpacity>
              );
            })}
          </ScrollView>
        </View>
      </Modal>
      {/* <TouchableOpacity
                style={{
                  backgroundColor: '#d4d4d4',
                  padding: 10,
                  marginTop: 0,
                }}
                onPress={() => setModalVisible(t => !t)}>
                <Text
                  style={{
                    color: 'black',
                    textAlign: 'center',
                  }}>
                  {data[modalValue-1]?.title}
                </Text>
              </TouchableOpacity> */}
      <View
        style={{
          //   marginHorizontal: 3,
          flexDirection: 'column',
          //   alignItems: 'center',
          flex: 7,
        }}>
        <Button
          title={
            firstTime
              ? 'Unselected'
              : newCropValue == ''
              ? data[modalValue - 1]?.title
              : newCropValue
          }
          onPress={() => setModalVisible(t => !t)}
          color={'green'}></Button>
        <Text
          style={{
            color: 'black',
            fontSize: 12,
          }}>
          Select Crop
        </Text>
      </View>
    </>
  );
}
```

FormCameraHandle.tsx

```
import {
  Text,
  BackHandler,
  Modal,
  View,
  TouchableOpacity,
  Image,
  Alert,
  PermissionsAndroid,
  Button,
  ActivityIndicator,
  Platform,
} from 'react-native';
import {useCallback, useEffect, useRef, useState} from 'react';
import {launchCamera, launchImageLibrary} from 'react-native-image-picker';
import {useDispatch, useSelector} from 'react-redux';
import {
  addImage,
  removeImage,
  selectBearingToCenter,
  selectDataCollection,
  selectDistanceToCenter,
  selectImagesList,
  selectLandCoverType,
  selectPrimaryCrop,
  selectSecondaryCrop,
} from '../features/DataCollectionSlice';
import {ScrollView} from 'react-native-gesture-handler';
import Marker, {
  ImageFormat,
  Position,
  TextBackgroundType,
} from 'react-native-image-marker';
import {CameraRoll} from '@react-native-camera-roll/camera-roll';
import {
  clearLocation,
  selectLocation,
  setLocation,
} from '../features/LocationSlice';
import {calculateExactLocation} from '../location/getLocation';
import {upload} from '../networking';

async function hasAndroidPermission() {
  const getCheckPermissionPromise = () => {
    // @ts-ignore
    if (Platform.Version >= 30) {
      return Promise.all([
        PermissionsAndroid.check(
          PermissionsAndroid.PERMISSIONS.READ_MEDIA_IMAGES,
        ),
        PermissionsAndroid.check(
          PermissionsAndroid.PERMISSIONS.READ_MEDIA_VIDEO,
        ),
      ]).then(
        ([hasReadMediaImagesPermission, hasReadMediaVideoPermission]) =>
          hasReadMediaImagesPermission && hasReadMediaVideoPermission,
      );
    } else {
      return PermissionsAndroid.check(
        PermissionsAndroid.PERMISSIONS.READ_EXTERNAL_STORAGE,
      );
    }
  };

  const hasPermission = await getCheckPermissionPromise();
  if (hasPermission) {
    return true;
  }
  const getRequestPermissionPromise = () => {
    // @ts-ignore
    if (Platform.Version >= 33) {
      return PermissionsAndroid.requestMultiple([
        PermissionsAndroid.PERMISSIONS.READ_MEDIA_IMAGES,
        PermissionsAndroid.PERMISSIONS.READ_MEDIA_VIDEO,
      ]).then(
        statuses =>
          statuses[PermissionsAndroid.PERMISSIONS.READ_MEDIA_IMAGES] ===
            PermissionsAndroid.RESULTS.GRANTED &&
          statuses[PermissionsAndroid.PERMISSIONS.READ_MEDIA_VIDEO] ===
            PermissionsAndroid.RESULTS.GRANTED,
      );
    } else {
      return PermissionsAndroid.request(
        PermissionsAndroid.PERMISSIONS.READ_EXTERNAL_STORAGE,
      ).then(status => status === PermissionsAndroid.RESULTS.GRANTED);
    }
  };

  return await getRequestPermissionPromise();
}

const requestCameraPermission = async () => {
  try {
    const granted = await PermissionsAndroid.request(
      PermissionsAndroid.PERMISSIONS.CAMERA,
      {
        title: 'The app needs the permissions to access your camera',
        message:
          'The camera permission is required to be able to click the photo ',
        buttonNeutral: 'Ask Me Later',
        buttonNegative: 'Cancel',
        buttonPositive: 'OK',
      },
    );
    if (granted === PermissionsAndroid.RESULTS.GRANTED) {
      console.log('You can use the camera');
    } else {
      console.log('Camera permission denied');
    }
  } catch (err) {
    console.warn(err);
  }
};

const imageProcessing = async (
  imageUri: string,
  primaryCrop: string,
  secondaryCrop: string,
  latitude: number,
  longitude: number,
  username: string,
  landType: string,
  completedTask: (newUri: string) => void,
  bearingToCenter: number,
  distanceToCenter: number,
) => {
  // if (!cropName || cropName == '') {
  //   Alert.alert(
  //     'Enter the crop name',
  //     "Type 0 to confirm if there's no crop to put in the field",
  //   );
  //   return null;
  // }
  // console.log(cropDetails);
  console.log('inside of image processor');
  // let finalCropsNamesString = "";
  if (landType == 'Cropland') {
    primaryCrop = primaryCrop.trim();
    secondaryCrop = secondaryCrop.trim();
  }
  // for(let a of cropDetails){
  //   finalCropsNamesString += `${a.name}:${a.crop}, `;
  // }
  // finalCropsNamesString = finalCropsNamesString.substring(0, finalCropsNamesString.length - 2);
  if (distanceToCenter != null && bearingToCenter != null) {
    const newLocation = calculateExactLocation(
      latitude,
      longitude,
      distanceToCenter,
      bearingToCenter,
    );
    latitude = newLocation.latitude;
    longitude = newLocation.longitude;
  }

  const options = {
    // background image
    backgroundImage: {
      src: {uri: imageUri},
      scale: 1,
    },
    watermarkTexts: [
      {
        text:
          landType == 'Cropland'
            ? `Latitude: ${latitude} \nLatitude: ${longitude}\nSeason 1: ${primaryCrop}\nSeason 2: ${secondaryCrop}\n${new Date()}`
            : `Latitude: ${latitude} \nLatitude: ${longitude}\nLand Cover Type: ${landType}\n${new Date()}`,
        position: {
          position: Position.topLeft,
        },
        style: {
          color: '#ffffff',
          fontSize: 30,
          fontName: 'Arial',
          shadowStyle: {
            dx: 0,
            dy: 0,
            radius: 15,
            color: '#000000',
          },
          textBackgroundStyle: {
            padding: '0% 1%',
            type: TextBackgroundType.none,
            color: '#000000',
          },
        },
      },
    ],
    scale: 1,
    quality: 100,
    filename: `${username}-${landType}-${new Date()}`,
    saveFormat: ImageFormat.jpg,
  };
  const permimssionPass = await hasAndroidPermission();
  console.log(permimssionPass);
  const path = await Marker.markText(options);
  const resUri = 'file:' + path;
  const newUri = await CameraRoll.saveAsset(resUri, {
    type: 'auto',
    album: 'geotagged photos',
  });
  completedTask(newUri.node.image.uri);
  // upload(newUri.node.image.uri)
};
export default function () {
  const imageList = useSelector(selectImagesList);
  const dispatch = useDispatch();
  const locationData = useSelector(selectLocation);
  const [isProcessingImage, setIsProcessingImage] = useState(false);
  const landCoverType = useSelector(selectLandCoverType);
  const wholeData = useSelector(selectDataCollection);
  const primaryCrop = useSelector(selectPrimaryCrop);
  const bearingToCenter = useSelector(selectBearingToCenter);
  const distanceToCenter = useSelector(selectDistanceToCenter);
  const secondaryCrop = useSelector(selectSecondaryCrop);
  //retrieve the actual username from here
  const username = 'dummy';
  useEffect(() => {
    console.log(locationData);
  }, [locationData, username, landCoverType]);
  const openCamera = useCallback(async () => {
    if (!locationData.latitude || !landCoverType) {
      Alert.alert(
        'Fill the previous fields of location beforehand',
        "Make sure you've filled land cover type, crop name(if applicable), and you've captured the location.",
      );
      return;
    }
    await requestCameraPermission();
    if (
      landCoverType == 'Cropland' &&
      (primaryCrop == null || secondaryCrop == null)
    ) {
      Alert.alert("the crops weren't set");
      return null;
    }
    if (!locationData.latitude) {
      Alert.alert(
        "The location wasn't set, please set it.",
        'Set the location',
      );
      return null;
    }
    const result = await launchCamera({mediaType: 'photo'}, (res: any) => {
      // let cropDetails:Object[] = []
      // if(wholeData.landCoverType == "Cropland"){
      //   console.log(wholeData)
      //   cropDetails.push({
      //     name: "Primary Crop",
      //     crop: wholeData.cropInformation.primaryCrop
      //   })
      //   cropDetails.push({
      //     name: "Secondary Crop",
      //     crop: wholeData.cropInformation.secondaryCrop
      //   })
      //   cropDetails = [...cropDetails, ...(wholeData.cropInformation.additionalSeasons.filter((value: any) => {
      //     if(value.name == null) return false;
      //     return true;
      //   }))];
      // }
      try {
        console.log(locationData);

        imageProcessing(
          res.assets[0].uri,
          primaryCrop,
          secondaryCrop,
          locationData.latitude,
          locationData.longitude,
          username,
          landCoverType,
          newUri => {
            setIsProcessingImage(false);
            dispatch(addImage(newUri));
          },
          bearingToCenter,
          distanceToCenter,
        );
        setIsProcessingImage(true);
      } catch (e) {
        console.log('Image saving rejected', e);
      }
    });
  }, [locationData, wholeData, username, landCoverType]);
  return (
    <>
      <View
        style={{
          marginTop: 10,
        }}>
        <ScrollView horizontal={true}>
          {imageList.map((value: string, id: number) => {
            if (value == null) return null;
            return (
              <View>
                <Image
                  source={{uri: value}}
                  key={id}
                  style={{
                    width: 150,
                    height: 150,
                    resizeMode: 'contain',
                    marginRight: 5,
                  }}></Image>
                <TouchableOpacity
                  style={{
                    position: 'absolute',
                    backgroundColor: 'white',
                    width: 20,
                    padding: 4,
                    borderRadius: 10,
                  }}
                  onPress={() => {
                    Alert.alert(
                      'Are you sure you want to delete this image?',
                      "Once deleted you can't recover it.",
                      [
                        {
                          text: 'Cancel',
                          onPress: () => {
                            console.log('delete operation canceled.');
                          },
                          //   style: 'cancel',
                        },
                        {
                          text: 'OK',
                          onPress: () => dispatch(removeImage(value)),
                        },
                      ],
                    );
                  }}>
                  <Text
                    style={{color: 'red', width: '100%', textAlign: 'center'}}>
                    X
                  </Text>
                </TouchableOpacity>
              </View>
            );
          })}
        </ScrollView>
        <View
          style={{
            flexDirection: 'row',
            // paddingHorizontal: 20,
            // marginVertical: 10,
            alignItems: 'center',
            justifyContent: 'center',
            paddingTop: 7,
          }}>
          {/* <TouchableOpacity
            onPress={() => {
              openCamera();
            }}
            style={{
              flex: 3,
              marginRight: 2,
              backgroundColor: 'grey',
              padding: 10,
              borderRadius: 20,
            }}>
            <Text>TAKE PICTURE</Text>
          </TouchableOpacity> */}
          <Button
            title="Take Picture"
            onPress={() => {
              openCamera();
            }}></Button>

          {/* The processing modal */}
          <Modal
            visible={isProcessingImage}
            style={{
              width: '100%',
              height: '100%',
              backgroundColor: 'white',
              zIndex: 10,
              position: 'absolute',
              flexDirection: 'column',
              alignItems: 'center',
              justifyContent: 'center',
            }}>
            <View
              style={{
                flexDirection: 'row',
                alignItems: 'center',
                justifyContent: 'center',
                width: '100%',
                height: '100%',
              }}>
              <ActivityIndicator
                style={{
                  height: 100,
                  width: 100,
                  flex: 1,
                }}
                color={'lightgreen'}
                size={'large'}
              />
              <Text
                // @ts-ignore
                style={{
                  color: 'black',
                  // width:"100%",
                  fontSize: 18,
                  fontWeight: 700,
                  flex: 3,
                }}>
                Applying the tag...
              </Text>
            </View>
          </Modal>
        </View>
      </View>
    </>
  );
}
```

LandingPage.tsx

```
import {
  Pressable,
  TouchableOpacity,
  Text,
  View,
  ActivityIndicator,
} from 'react-native';
import Button from './Button';
import {ScrollView} from 'react-native-gesture-handler';
import SmallDrawer from './SmallDrawer';
import {useDispatch, useSelector} from 'react-redux';
import {selectWaterSourceType} from '../features/DataCollectionSlice';
import {useEffect, useState} from 'react';
import {getBottomIndexCount, getJwt, storage} from '../localStorage';
import {useMMKVListener, useMMKVString} from 'react-native-mmkv';
import {BACKEND_URL, upload} from '../networking';
import axios from 'axios';
import {
  selectUiData,
  setWaterSourceCropTypeLandCover,
} from '../features/UISlice';
// import { TouchableOpacity } from "react-native-gesture-handler";
export interface MessageProp {
  show: boolean;
  message1: string;
  message2: string;
}

export default function HomeScreen({navigation}: {navigation: any}) {
  const [email, setEmail] = useMMKVString('user.email');
  const [jwt, setJwt] = useMMKVString('user.jwt');
  const dispatch = useDispatch();
  // at any saving of the  data, and not the initial one.
  useMMKVListener(key => {
    const keys = storage.getAllKeys();
    setUnsynced(storage.getNumber('counter') || 0);
    let obj = [];
    for (let a of keys) {
      obj.push(storage.getString(a));
    }
    setData(obj);
    console.log(`Value for "${key}" changed!`);
  });
  const waterSource = useSelector(selectWaterSourceType);
  console.log(waterSource);
  const [data, setData] = useState<any>([]);
  const [unsynced, setUnsynced] = useState<any>(0);
  const [synced, setSynced] = useState<Number>(-1);
  const uiData = useSelector(selectUiData);
  const [disabled, setDisabled] = useState<boolean>(false);
  const [logMessage, setLogMessage] = useState<MessageProp>({
    message1: '',
    message2: '',
    show: false,
  });
  // at first mount
  useEffect(() => {
    const count = storage.getNumber('counter');
    const jwt = getJwt();
    const bottomIndex = storage.getNumber('bottomIndex');
    async function syncSetter() {
      const response = await axios.post(
        BACKEND_URL + 'api/v1/sync/synced/',
        {},
        {
          headers: {
            Authorization: `Bearer ${jwt}`,
          },
        },
      );
      console.log(response.data.synced, 'IS THE SYNCED AMOUNT');
      setSynced(response.data.synced);
    }
    syncSetter();
    dispatch(setWaterSourceCropTypeLandCover());
    if (typeof count == 'number' && typeof bottomIndex == 'number') {
      console.log('bottomIndex', bottomIndex);
      console.log('counter', count);
      const result = count - bottomIndex + 1;
      setUnsynced(result || 0);
    } else {
      console.log('setting unsynced failed FAILED');
    }
  }, [data]);
  useEffect(() => {
    const keys = storage.getAllKeys();
    let obj = [];
    for (let a of keys) {
      obj.push(storage.getString(a));
    }
    setData(obj);
  }, []);
  // put this inside of the form
  return (
    <ScrollView style={{flex: 1, flexDirection: 'column', marginTop: 10}}>
      <Button handler={() => navigation.navigate('datacollection')}>
        COLLECT DATA
      </Button>
      <Button
        handler={() => {
          if (disabled) return;
          setDisabled(true);
          upload(() => setDisabled(false), setLogMessage);
          // dispatch(setWaterSourceCropTypeLandCover());
        }}>
        SYNC DATA
      </Button>
      <SmallDrawer
        style={{
          backgroundColor: '#888484',
          width: '90%',
          padding: 10,
        }}
        title={'Sync Data'}
        data={[
          {
            value: synced,
            title: 'Synced',
          },
          {
            value: unsynced,
            title: 'Not Synced',
          },
        ]}></SmallDrawer>

      {logMessage.show ? (
        <View
          style={{
            flex: 1,
            flexDirection: 'row',
            justifyContent: 'space-between',
            paddingHorizontal: 20,
            padding: 5,
            borderWidth: 1,
            marginTop: 4,
            borderColor: 'green',
            borderRadius: 10,
            marginHorizontal: 10,
          }}>
          <View>
            <Text
              style={{
                color: 'black',
              }}>
              {logMessage.message1}
            </Text>
            <Text
              style={{
                color: 'black',
                paddingHorizontal: 5,
                padding: 5,
              }}>
              - {logMessage.message2}
            </Text>
          </View>
          {logMessage.message1 != 'SYNC COMPLETED' ? (
            <ActivityIndicator size={'large'} color={'red'} />
          ) : null}
        </View>
      ) : null}

      <SmallDrawer
        style={{
          backgroundColor: '#888484',
          width: '90%',
          padding: 10,
          marginTop: 30,
        }}
        title="Water Source"
        data={uiData.waterSource}></SmallDrawer>
      <SmallDrawer
        style={{
          backgroundColor: '#888484',
          width: '90%',
          padding: 10,
          marginTop: 3,
        }}
        title="Crop Type"
        data={uiData.cropType}></SmallDrawer>
      <SmallDrawer
        style={{
          backgroundColor: '#888484',
          width: '90%',
          padding: 10,
          marginTop: 3,
        }}
        title="Land Cover Type"
        data={uiData.landCover}></SmallDrawer>
    </ScrollView>
  );
}
```

MapChooseLocation.tsx

```
import Geolocation from '@react-native-community/geolocation';
import {useCallback, useEffect, useRef, useState} from 'react';
import {
  Alert,
  Modal,
  StyleSheet,
  Text,
  View,
  TouchableOpacity,
  Button,
} from 'react-native';
import MapView, {
  Callout,
  LatLng,
  MapPressEvent,
  MapType,
  Marker,
  MarkerDragStartEndEvent,
  PROVIDER_GOOGLE,
  Region,
} from 'react-native-maps'; // remove PROVIDER_GOOGLE import if not using Google Maps
import {Position, PositionError} from '../types';
import {useNetInfo} from '@react-native-community/netinfo';

// import {  } from 'react-native-gesture-handler';
// import {  } from 'react-native-reanimated/lib/typescript/Animated';

const styles = StyleSheet.create({
  container: {
    ...StyleSheet.absoluteFillObject,
    height: '100%',
    width: '100%',
    justifyContent: 'flex-end',
    alignItems: 'center',
  },
  map: {
    ...StyleSheet.absoluteFillObject,
  },
});

const MapChooser = ({
  handler,
}: {
  handler: (latitude: number, longitude: number) => void;
}) => {
  const [region, setRegion] = useState<Region>({
    longitude: 0,
    latitude: 0,
    latitudeDelta: 0.0015,
    longitudeDelta: 0.0015,
  });
  const [mapType, setMapType] = useState<MapType>('satellite');
  const [mapCoordinates, setMapCoordinates] = useState<LatLng>({
    latitude: region.latitude,
    longitude: region.longitude,
  });
  const ref = useRef<MapView>();

  return (
    <View style={styles.container}>
      <MapView
        mapType={mapType}
        provider={PROVIDER_GOOGLE} // remove if not using Google Maps
        style={styles.map}
        region={region}
        loadingEnabled={true}
        loadingIndicatorColor={'grey'}
        showsUserLocation={true}
        showsMyLocationButton={true}
        onPress={(event: MapPressEvent) => {
          const coordinates = event.nativeEvent.coordinate;
          setMapCoordinates({
            latitude: coordinates.latitude,
            longitude: coordinates.longitude,
          });
        }}
        onMapReady={() => {
          Geolocation.setRNConfiguration({
            skipPermissionRequests: false,
            authorizationLevel: 'auto',
            enableBackgroundLocationUpdates: true,
            locationProvider: 'auto',
          });
          Geolocation.getCurrentPosition(
            (position: Position) => {
              console.log(position);
              setRegion({
                latitude: position.coords.latitude,
                longitude: position.coords.longitude,
                latitudeDelta: 0.0015,
                longitudeDelta: 0.0015,
              });
              setMapCoordinates({
                latitude: position.coords.latitude,
                longitude: position.coords.longitude,
              });
            },
            (error: PositionError) => console.log(error),
            {},
          );
        }}>
        <Marker
          coordinate={mapCoordinates}
          draggable
          tappable
          onDragEnd={(event: MarkerDragStartEndEvent) => {
            const coordinates = event.nativeEvent.coordinate;
            setMapCoordinates({
              latitude: coordinates.latitude,
              longitude: coordinates.longitude,
            });
          }}>
          <Callout tooltip={false} style={{borderRadius: 10}}>
            <Text style={{color: 'blue'}}>
              Place this to the position you want to mark.
            </Text>
          </Callout>
        </Marker>
      </MapView>
      <View
        style={{
          position: 'absolute',
          width: '100%',
          height: 'auto',
        }}>
        <View
          style={{
            flexDirection: 'row',
            margin: 10,
            justifyContent: 'space-between',
          }}>
          <TouchableOpacity
            onPress={() => {
              setMapType((value) => {
                if(value == "satellite"){
                  return "standard";
                }
                else return "satellite"
              })
            }}
            style={{
              backgroundColor: 'lightgreen',
              padding: 10,
              paddingHorizontal: 15,
              borderRadius: 15,
            }}>
            <Text
              // @ts-ignore
              style={{
                fontSize: 18,
                color:"black"
              }}>
              View: {mapType} map
            </Text>
          </TouchableOpacity>
          <TouchableOpacity
            onPress={() => {
              handler(mapCoordinates.latitude, mapCoordinates.longitude);
            }}
            style={{
              backgroundColor: 'green',
              padding: 10,
              paddingHorizontal: 15,
              borderRadius: 15,
            }}>
            <Text
              // @ts-ignore
              style={{
                fontSize: 18,
                fontWeight: 900,
              }}>
              Done
            </Text>
          </TouchableOpacity>
        </View>
        <View
          style={{
            backgroundColor: 'turquoise',
            padding: 10,
            borderRadius: 20,
          }}>
          <Text
            style={{
              color: 'black',
              margin: 10,
              fontSize: 14,
              textAlign: 'center',
            }}>
            Tap at any location, or hold and drag the marker above to mark and
            get the coordinates of the position
          </Text>
          <Text
            style={{color: 'red', textAlign: 'center', marginHorizontal: 10}}>
            Latitude: {mapCoordinates.latitude}
          </Text>
          <Text
            style={{color: 'red', textAlign: 'center', marginHorizontal: 10}}>
            Longitude: {mapCoordinates.longitude}
          </Text>
        </View>
      </View>
    </View>
  );
};
// handler is for successful operations and closer for cleaning up things that shouldn't have been happening
export default function ({
  handler,
  closer,
}: {
  handler: (latitude: number, longitude: number) => void;
  closer: () => void;
}) {
  const [isRendered, setIsRendered] = useState(false);
  const {type, isConnected} = useNetInfo();
  useEffect(() => {
    if (type != 'none' && isConnected == true) {
      setIsRendered(true);
    }
    if (type == 'none' || isConnected == false) {
      Alert.alert(
        "You're not connected to the Internet.",
        'Connect to the internet to use the maps functionality',
      );
      if (isRendered == false) {
        closer();
      }
    }
    console.log(type, isConnected);
  }, [type, isConnected]);
  if (isRendered)
    return (
      <>
        <MapChooser handler={handler} />
      </>
    );
  return null;
}
```

SmallDrawer.tsx

```
import React, {useState} from 'react';
import {
  TouchableOpacity,
  Text,
  View,
  StyleSheet,
  StyleSheetProperties,
  StyleProp,
  ViewStyle,
  ScrollView,
} from 'react-native';
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
} from 'react-native-reanimated';

type dataItem = {
  value: number;
  title: string;
};

export default function MyComponent({
  title,
  data,
  style,
}: {
  title: string;
  data: dataItem[];
  style: StyleProp<ViewStyle>;
}) {
  const [isVisible, setIsVisible] = useState(false);
  const height = useSharedValue(0);

  const toggleVisibility = () => {
    setIsVisible(prev => !prev);
    height.value = withSpring(isVisible ? 0 : 100, {
      damping: 10,
      stiffness: 100,
    }); // Adjust damping and stiffness for desired animation
  };

  const animatedStyle = useAnimatedStyle(() => {
    return {
      maxHeight: height.value,
      overflow: 'hidden',
    };
  });

  return (
    <View style={styles.container}>
      <TouchableOpacity style={style} onPress={toggleVisibility}>
        <Text
          // @ts-ignore
          style={{
            fontWeight: 700,
          }}>
          {title}
        </Text>
      </TouchableOpacity>

      <Animated.View style={[styles.content, animatedStyle]}>
        <ScrollView>
          {data.map((value: any, id: number) => {
            return (
              <>
                <TouchableOpacity
                  key={id}
                  style={{
                    flexDirection: 'row',
                    margin: 40,
                    marginVertical: 5,
                    width: '90%',
                  }}>
                  <Text
                    style={{
                      flex: 1,
                      color: 'black',
                    }}>
                    {value.value}
                  </Text>
                  <Text
                    style={{
                      flex: 1,
                      color: 'black',
                    }}>
                    {value.title}
                  </Text>
                </TouchableOpacity>
              </>
            );
          })}
        </ScrollView>
      </Animated.View>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
  button: {
    color: 'black',
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 20,
  },
  content: {
    width: '100%',
  },
  text: {
    color: 'black',
    fontSize: 16,
    textAlign: 'center',
    marginTop: 10,
  },
});
```

- The `datacollection` folder has all the files for the datacollection form, with `Main` being the entrypoint
  . We descrive it's code below

CCE.tsx

```
import {useEffect, useState} from 'react';
import {Button, Text, TextInput, View} from 'react-native';
import CustomDatePicker from '../CustomDatePicker';
import {useDispatch, useSelector} from 'react-redux';
import {
  selectCCEData,
  setBiomassWeight,
  setCCECaptured,
  setCCEHarvestDate,
  setCCESowDate,
  setCultivar,
  setGrainWeight,
  setXSampleSize,
  setYSampleSize,
} from '../../features/DataCollectionSlice';

export default function () {
  const [sowDate, setSowDate] = useState(new Date());
  const [harvestDate, setHarvestDate] = useState(new Date());
  const dispatch = useDispatch();
  const CCEData: {
    isCaputred: boolean;
    sampleSize: {
      x: string;
      y: string;
    };
    grainWeight: string;
    biomassWeight: string;
    cultivar: string;
    sowDate: Date;
    harvestDate: Date;
  } = useSelector(selectCCEData);
  // being lazy in here, i am not propogating the changes in state indirectly, directly updating and fetching the redux state, i am sorry here.
  // if(CCEData == undefined) return null
  useEffect(() => {
    if (
      (CCEData.sampleSize.x != null) &&
      (CCEData.sampleSize.y != null) &&
      (CCEData.grainWeight != null) &&
      (CCEData.biomassWeight != null) &&
      (CCEData.cultivar != null) &&
      (CCEData.sowDate != null) &&
      (CCEData.harvestDate != null)
    ) {
      if (CCEData.isCaputred != true) {
        dispatch(setCCECaptured(true));
      }
    } else {
      dispatch(setCCECaptured(false));
    }
  }, [CCEData]);
  return (
    <>
      <View
        style={{
          flexDirection: 'row',
          alignItems: 'center',
          justifyContent: 'center',
          //   width:"100%",
          marginHorizontal: 25,
          marginTop: 5,
        }}>
        <Text
          style={{
            color: 'black',
            flex: 6,
            // padding:5
          }}>
          Sample Size (in metre X metre):
        </Text>
        <TextInput
          allowFontScaling={true}
          blurOnSubmit={true}
          autoFocus={true}
          inputMode="decimal"
          value={CCEData?.sampleSize.x}
          onChangeText={value => {
            dispatch(setXSampleSize(value));
          }}
          style={{
            backgroundColor: 'white',
            flex: 1,
            paddingHorizontal: 10,
            color: 'black',
            marginHorizontal: 5,
            borderWidth: 1,
            borderColor: 'grey',
            borderRadius: 14,
          }}></TextInput>
        <Text>X</Text>
        <TextInput
          allowFontScaling={true}
          blurOnSubmit={true}
          // autoFocus={true}
          inputMode="decimal"
          value={CCEData?.sampleSize.y}
          onChangeText={value => {
            dispatch(setYSampleSize(value));
          }}
          style={{
            backgroundColor: 'white',
            flex: 1,
            paddingHorizontal: 10,
            color: 'black',
            marginHorizontal: 5,
            borderWidth: 1,
            borderColor: 'grey',
            borderRadius: 14,
          }}></TextInput>
      </View>
      <View
        style={{
          flexDirection: 'row',
          alignItems: 'center',
          justifyContent: 'center',
          //   width:"100%",
          marginHorizontal: 25,
          marginTop: 5,
        }}>
        <Text
          style={{
            color: 'black',
            flex: 6,
            // padding:5
          }}>
          Grain Weight (in kilograms):
        </Text>

        <TextInput
          allowFontScaling={true}
          blurOnSubmit={true}
          inputMode="decimal"
          style={{
            backgroundColor: 'white',
            flex: 1,
            paddingHorizontal: 10,
            marginHorizontal: 5,
            borderWidth: 1,
            color: 'black',
            borderColor: 'grey',
            borderRadius: 14,
          }}
          value={CCEData.grainWeight}
          onChangeText={value => {
            dispatch(setGrainWeight(value));
          }}></TextInput>
      </View>
      <View
        style={{
          flexDirection: 'row',
          alignItems: 'center',
          justifyContent: 'center',
          //   width:"100%",
          marginHorizontal: 25,
          marginTop: 5,
        }}>
        <Text
          style={{
            color: 'black',
            flex: 6,
            // padding:5
          }}>
          Bio-Mass Weight (in kilograms):
        </Text>

        <TextInput
          allowFontScaling={true}
          blurOnSubmit={true}
          inputMode="decimal"
          value={CCEData.biomassWeight}
          onChangeText={value => {
            dispatch(setBiomassWeight(value));
          }}
          style={{
            backgroundColor: 'white',
            flex: 1,
            paddingHorizontal: 10,
            marginHorizontal: 5,
            borderWidth: 1,
            color: 'black',
            borderColor: 'grey',
            borderRadius: 14,
          }}></TextInput>
      </View>
      <View
        style={{
          flexDirection: 'row',
          alignItems: 'center',
          justifyContent: 'center',
          //   width:"100%",
          marginHorizontal: 25,
          marginTop: 5,
        }}>
        <Text
          style={{
            color: 'black',
            flex: 6,
            // padding:5
          }}>
          Cultivar:
        </Text>
        <TextInput
          allowFontScaling={true}
          blurOnSubmit={true}
          inputMode="text"
          style={{
            backgroundColor: 'white',
            flex: 3,
            paddingHorizontal: 10,
            marginHorizontal: 5,
            borderWidth: 1,
            color: 'black',
            borderColor: 'grey',
            borderRadius: 14,
          }}
          value={CCEData.cultivar}
          onChangeText={value => {
            dispatch(setCultivar(value));
          }}></TextInput>
      </View>
      <View
        style={{
          flexDirection: 'row',
          alignItems: 'center',
          justifyContent: 'center',
          //   width:"100%",
          marginHorizontal: 25,
          marginTop: 5,
        }}>
        <Text
          style={{
            color: 'black',
            flex: 6,
            // padding:5
          }}>
          Sowing Date:
        </Text>
        <CustomDatePicker
          date={sowDate}
          setDate={(date: Date) => {
            setSowDate(date);
            //also dispatch the state here, it is important for the redux state to stay null at first thats why
            dispatch(setCCESowDate(date.toString()));
          }}
        />
      </View>
      <View
        style={{
          flexDirection: 'row',
          alignItems: 'center',
          justifyContent: 'center',
          //   width:"100%",
          marginHorizontal: 25,
          marginTop: 5,
        }}>
        <Text
          style={{
            color: 'black',
            flex: 6,
            // padding:5
          }}>
          Harvest Date:
        </Text>
        <CustomDatePicker
          date={harvestDate}
          setDate={(date: Date) => {
            setHarvestDate(date);
            dispatch(setCCEHarvestDate(date.toString()));
          }}
        />
      </View>
    </>
  );
}

```

CropInformation.tsx

```
import {Text, TextInput, View} from 'react-native';
import CustomModal from '../CustomModal';
import {
  cropGrowthStageData,
  cropIntensityData,
  croppingPatternData,
  cropsData,
  livestockData,
  waterSourceData,
} from '../../data';
import {useDispatch} from 'react-redux';
import {
  setCropGrowthStage,
  setCropIntensity,
  setCropRemarks,
  setCroppingPattern,
  setLandCoverType,
  // setLiveStock,
  setPrimaryCrop,
  setSecondaryCrop,
  setWaterSource,
} from '../../features/DataCollectionSlice';
import SeasonSelection from './SeasonSelection';

export default function () {
  const dispatch = useDispatch();
  return (
    <>
      <View
        style={{
          marginTop: 15,
          flexDirection: 'column',
          // marginHorizontal: 20,
        }}>
        <Text
          style={{
            color: 'black',
            backgroundColor: '#888484',
            padding: 5,
            marginHorizontal: 20,
          }}>
          Crop Information
        </Text>
        <View
          style={{
            flexDirection: 'row',
            alignItems: 'center',
            justifyContent: 'center',
            //   width:"100%",
            marginHorizontal: 25,
            marginTop: 5,
          }}>
          <Text
            style={{
              color: 'black',
              flex: 6,
              // padding:5
            }}>
            Water Source
          </Text>
          <CustomModal
            data={waterSourceData}
            action={payload => dispatch(setWaterSource(payload))}></CustomModal>
        </View>
        <View
          style={{
            flexDirection: 'row',
            alignItems: 'center',
            justifyContent: 'center',
            //   width:"100%",
            marginHorizontal: 25,
            marginTop: 5,
          }}>
          <Text
            style={{
              color: 'black',
              flex: 6,
              // padding:5
            }}>
            Crop Intensity
          </Text>

          <CustomModal
            data={cropIntensityData}
            action={payload =>
              dispatch(setCropIntensity(payload))
            }></CustomModal>
        </View>

        <SeasonSelection></SeasonSelection>

        <View
          style={{
            flexDirection: 'row',
            alignItems: 'center',
            justifyContent: 'center',
            //   width:"100%",
            marginHorizontal: 25,
            marginTop: 5,
          }}>
          <Text
            style={{
              color: 'black',
              flex: 6,
              // padding:5
            }}>
            Cropping Pattern
          </Text>

          <CustomModal
            data={croppingPatternData}
            action={payload =>
              dispatch(setCroppingPattern(payload))
            }></CustomModal>
        </View>
        <View
          style={{
            flexDirection: 'row',
            alignItems: 'center',
            justifyContent: 'center',
            //   width:"100%",
            marginHorizontal: 25,
            marginTop: 5,
          }}>
          <Text
            style={{
              color: 'black',
              flex: 6,
              // padding:5
            }}>
            Crop Growth Stage
          </Text>

          <CustomModal
            data={cropGrowthStageData}
            action={payload =>
              dispatch(setCropGrowthStage(payload))
            }></CustomModal>
        </View>
        {/* <View
          style={{
            flexDirection: 'row',
            alignItems: 'center',
            justifyContent: 'center',
            //   width:"100%",
            marginHorizontal: 25,
            marginTop: 5,
          }}>
          <Text
            style={{
              color: 'black',
              flex: 6,
              // padding:5
            }}>
            Live Stock
          </Text>

          <CustomModal
            data={livestockData}
            action={payload => dispatch(setLiveStock(payload))}></CustomModal>
        </View> */}

        <View
          style={{
            flexDirection: 'column',
            alignItems: 'center',
            justifyContent: 'center',
            //   width:"100%",
            marginHorizontal: 25,
            marginTop: 5,
          }}>
          <Text
            style={{
              color: 'black',
              flex: 6,
              textAlign: 'left',
              width: '100%',
              // padding:5
            }}>
            Remarks and notes:
          </Text>
          <TextInput
            multiline={true}
            numberOfLines={3}
            style={{
              backgroundColor: '#eef7eb',
              width: '100%',
              padding: 10,
              textAlignVertical: 'top',
              borderWidth: 1,
              color: 'black',
              borderColor: 'grey',
              borderRadius: 10,
              marginTop: 5,
            }}
            onChangeText={value => {
              dispatch(setCropRemarks(value));
            }}></TextInput>
        </View>
      </View>
    </>
  );
}

```

Description.tsx

```
import { Text, TextInput, View } from "react-native";
import { useDispatch } from "react-redux";
import { setLocationDescription } from "../../features/DataCollectionSlice";

export default function(){
  const dispatch = useDispatch()
    return (
        <View
            style={{
              marginTop: 15,
              flexDirection: 'column',
              // marginHorizontal: 20,
            }}>
            <Text
              style={{
                color: 'black',
                backgroundColor: '#888484',
                padding: 5,
                marginHorizontal: 20,
              }}>
              Description of location
            </Text>
        <TextInput style={{
            padding:10,
            margin:5,
            borderWidth:1,
            borderRadius:10,
            marginHorizontal:15,
            backgroundColor:"#eef7eb",
            color:"black",
            borderColor:"grey",
            fontSize:16,
            textAlignVertical:"top"
        }} multiline={true}
        onChangeText={(value) => {
            dispatch(setLocationDescription(value))
        }}
        numberOfLines={3}
        >

        </TextInput>
          </View>
    )
}
```

Location.tsx

```
import {useCallback, useState} from 'react';
import {
  Alert,
  Button,
  Modal,
  Switch,
  Text,
  TouchableOpacity,
  View,
} from 'react-native';
import {useDispatch, useSelector} from 'react-redux';
import {
  clearLocation,
  selectLocation,
  setLocation,
} from '../../features/LocationSlice';
import MapChooseLocation from '../MapChooseLocation';
import Geolocation from '@react-native-community/geolocation';
import {Position} from '../../types';
import { selectCapturedFromMap, setCapturedFromMap } from '../../features/DataCollectionSlice';

export default function () {
  const [isEnabled, setIsEnabled] = useState(false);
  const capturedFromMap = useSelector(selectCapturedFromMap);
  const locationData = useSelector(selectLocation);
  const dispatch = useDispatch();
  // const setCapturedFromMapBoolean = useCallback((bool: boolean) => {
  //   dispatch(setCapturedFromMap(bool))
  // }, [capturedFromMap]);
  const toggleSwitch = () => {
    setIsEnabled(t => {
      dispatch(setCapturedFromMap(!t));
      return !t
    });
  };

  const getCurrentPosition = useCallback(() => {
    Geolocation.setRNConfiguration({
      skipPermissionRequests: false,
      authorizationLevel: 'auto',
      enableBackgroundLocationUpdates: true,
      locationProvider: 'auto',
    });
    Geolocation.getCurrentPosition(
      (pos: Position) => {
        dispatch(setLocation(pos.coords));
        console.log(pos);
      },
      (error: any) =>
        Alert.alert('GetCurrentPosition Error', JSON.stringify(error)),
      {
        enableHighAccuracy: true, // don't worry, inside buildings
      },
    );
  }, []);
  const [isMapModalOpen, setIsMapModalOpen] = useState(false);
  const locationSetter = () => {
    if (isEnabled == false) {
      // getLocation();
      getCurrentPosition();
    } else {
      setIsMapModalOpen(true);
    }
  };
  return (
    <>
      <Text
        style={{
          color: 'black',
          backgroundColor: '#888484',
          padding: 5,
          marginHorizontal: 20,
        }}>
        Location
      </Text>

      <View
        style={{
          flexDirection: 'row',
          marginHorizontal: 20,
        }}>
        <Text
          style={{
            color: 'black',
            flex: 5,
            margin: 5,
          }}>
          Capture Location using Maps?
        </Text>
        <Switch
          trackColor={{false: '#767577', true: 'pink'}}
          thumbColor={isEnabled ? 'violet' : '#f4f3f4'}
          ios_backgroundColor="#3e3e3e"
          onValueChange={toggleSwitch}
          value={isEnabled}
        />
      </View>

      <View
        style={{
          flexDirection: 'row',
          marginHorizontal: 20,
          justifyContent: 'center',
          alignItems: 'center',
        }}>
        <Text
          style={{
            color: 'black',
            flex: 5,
            margin: 5,
          }}>
          Capture the location
        </Text>
        <Modal
          style={{
            width: '100%',
            height: '100%',
            backgroundColor: 'white',
          }}
          animationType="slide"
          onRequestClose={() => {
            setIsMapModalOpen(false);
          }}
          visible={isMapModalOpen}>
          <MapChooseLocation
            closer={() => {
              setIsMapModalOpen(false);
              if (!locationData.latitude) setIsEnabled(false); // in case the user opens the map after turning it on once, meaning the data is present from the past stuff.
            }}
            handler={(latitude, longitude) => {
              dispatch(
                setLocation({
                  latitude: latitude,
                  longitude: longitude,
                  accuracy: 10,
                }),
              );
              setIsMapModalOpen(false);
              setIsEnabled(true);
            }}
          />
        </Modal>
        <Button
          title="Capture Location"
          onPress={() => locationSetter()}></Button>
      </View>

      <View
        style={{
          flexDirection: 'row',
          alignItems: 'center',
          justifyContent: 'center',
          //   width:"100%",
          marginHorizontal: 25,
          marginTop: 5,
        }}>
        <Text
          style={{
            color: 'black',
            flex: 6,
            // padding:5
          }}>
          Accuracy Correction: {locationData.accuracy}
        </Text>
        <Button
          title="Clear"
          onPress={() => {
            dispatch(clearLocation());
            setIsEnabled(false);
          }}></Button>
      </View>

      <View
        style={{
          flexDirection: 'column',
          marginHorizontal: 20,
        }}>
        <Text
          style={{
            color: 'black',
            flex: 5,
            margin: 5,
          }}>
          Lat: {locationData.latitude}
        </Text>
        <Text
          style={{
            color: 'black',
            flex: 5,
            margin: 5,
          }}>
          Lon: {locationData.longitude}
        </Text>
      </View>
    </>
  );
}

```

LocationOffset.tsx

```
import { Slider } from "@miblanchard/react-native-slider";
import { useCallback, useEffect, useState } from "react";
import { Button, Text, ToastAndroid, TouchableOpacity, View } from "react-native";
// @ts-ignore
import CompassHeading from 'react-native-compass-heading';
import { useDispatch, useSelector } from "react-redux";
import { selectDegreesToNorth, selectResetter, setDegreesToNorth } from "../../features/LocationSlice";
import { hell, storage } from "../../localStorage";
import { selectCapturedFromMap, setBearingToCenterData, setDistanceToCenterData } from "../../features/DataCollectionSlice";
export default function(){

  const [distanceToCenter, setDistanceToCenter] = useState(70);

  const [bearingToCenter, setBearingToCenter] = useState<number | null>(null);
  // const [degreesToNorth, setDegreesToNorth] = useState(0);
  const resetterValue = useSelector(selectResetter);
  useEffect(() => {
    setDistanceToCenter(70)
    setBearingToCenter(null)
    console.log(resetterValue)
  }, [resetterValue])
  const degreesToNorth = useSelector(selectDegreesToNorth)
  const capturedFromMap = useSelector(selectCapturedFromMap);
  const showToastWithGravityAndOffset = () => {
    ToastAndroid.showWithGravityAndOffset(
      'Hold the phone steady for better accuracy',
      ToastAndroid.LONG,
      ToastAndroid.BOTTOM,
      25,
      50,
    );
  };
  const findBearingToCenter = useCallback(() => {
    showToastWithGravityAndOffset()
    setTimeout(() => {
      setBearingToCenter(degreesToNorth)
      dispatch(setBearingToCenterData(degreesToNorth))
    }, 1750)
  }, [degreesToNorth])
  const dispatch = useDispatch();
  useEffect(() => {
    const degree_update_rate = 3;
    let timerId = setTimeout(() => console.log("testing"), 10);
    CompassHeading.start(degree_update_rate, ({heading, accuracy}: {
      heading: number;
      accuracy: number;
    }) => {
      // console.log('CompassHeading: ', heading, accuracy);
      // setDegreesToNorth(heading);
      // clearTimeout(timerId)
      // timerId = setTimeout(() => dispatch(setDegreesToNorth(heading)), 0);
      dispatch(setDegreesToNorth(heading))
    });


    return () => {
      CompassHeading.stop();
    };
  }, []);
  if(capturedFromMap) return null;
  return (
    <>
        <View
            style={{
              marginTop: 15,
              flexDirection: 'column',
              // marginHorizontal: 20,
            }}>
            <Text
              style={{
                color: 'black',
                backgroundColor: '#888484',
                padding: 5,
                marginHorizontal: 20,
              }}>
              Location Offset
            </Text>

            <View
              style={{
                flexDirection: 'row',
                alignItems: 'center',
                justifyContent: 'center',
                //   width:"100%",
                marginHorizontal: 25,
                marginTop: 5,
              }}>
              <Text
                style={{
                  color: 'black',
                  flex: 6,
                  // padding:5
                }}>
                Bearing to Center: {bearingToCenter}
              </Text>
              {/* <TouchableOpacity
                style={{
                  backgroundColor: '#d4d4d4',
                  padding: 10,
                  marginTop: 4,
                }}>
                <Text
                  style={{
                    color: 'black',
                    // textAlign: 'center',
                  }}
                  onPress={() => findBearingToCenter()}
                  >
                  CAPTURE
                </Text>
              </TouchableOpacity> */}
              <Button title="Capture" onPress={() => findBearingToCenter()}></Button>
            </View>

            <View
              style={{
                flexDirection: 'column',
              }}>
              <Text
                style={{
                  color: 'black',
                  flex: 6,
                  marginHorizontal: 25,
                  marginTop: 10,
                  // padding:5
                }}>
                Distance to Center: {distanceToCenter} meters
              </Text>
              <Slider
                animateTransitions
                maximumValue={150}
                minimumValue={0}
                value={distanceToCenter}
                step={1}
                // @ts-ignore
                onValueChange={value => setDistanceToCenter(() => {
                  dispatch(setDistanceToCenterData(value))
                  return value
                })}
                containerStyle={{
                  marginHorizontal: 20,
                }}
              />
            </View>
          </View>
        </>
    )
}
```

Main.tsx

```
import {
  Alert,
  Modal,
  Pressable,
  ScrollView,
  Switch,
  Text,
  View,
  TouchableOpacity,
} from 'react-native';
import Button from '../Button';
import {useCallback, useRef, useState} from 'react';
import {Slider} from '@miblanchard/react-native-slider';
import CustomModal from '../CustomModal';
import {
  selectLandCoverType,
  setIsCapturedCCE,
  setLandCoverType,
} from '../../features/DataCollectionSlice';
import {useDispatch, useSelector} from 'react-redux';
import CropInformation from './CropInformation';
import {landData} from '../../data';
import FormCameraHandle from '../FormCameraHandle';
import {
  clearLocation,
  selectLocation,
  setLocation,
} from '../../features/LocationSlice';
import MapChooseLocation from '../MapChooseLocation';
import Geolocation from '@react-native-community/geolocation';
import CCE from './CCE';
import Location from './Location';
import LocationOffset from './LocationOffset';
import Description from './Description';
import QualityControl from './QualityControl';

export default function ({navigation}: {navigation: any}) {
  const locationData = useSelector(selectLocation);
  const [isCaptureCCE, setIsCaptureCCE] = useState(false);
  const dispatch = useDispatch();
  const scrollRef = useRef(null);
  const landCoverType = useSelector(selectLandCoverType);
  return (
    <>
      <ScrollView ref={scrollRef}>
        <View
          style={{
            marginTop: 5,
            flexDirection: 'column',
            // marginHorizontal: 20,
          }}>
          {/* The Location Section */}
          <Location />
          {/* Location Offset section */}
          <LocationOffset />
          {/* The location class section */}
          <View
            style={{
              marginTop: 15,
              flexDirection: 'column',
              // marginHorizontal: 20,
            }}>
            <Text
              style={{
                color: 'black',
                backgroundColor: '#888484',
                padding: 5,
                marginHorizontal: 20,
              }}>
              Location Class
            </Text>
            <View
              style={{
                flexDirection: 'row',
                alignItems: 'center',
                justifyContent: 'center',
                //   width:"100%",
                marginHorizontal: 25,
                marginTop: 5,
              }}>
              <Text
                style={{
                  color: 'black',
                  flex: 6,
                  // padding:5
                }}>
                Land Cover Type:
              </Text>
              <CustomModal
                data={landData}
                action={payload =>
                  dispatch(setLandCoverType(payload))
                }></CustomModal>
            </View>
          </View>

          {landCoverType == 'Cropland' ? <CropInformation /> : null}

          {/* CCE */}
          <View
            style={{
              marginTop: 15,
              flexDirection: 'row',
              // marginHorizontal: 20,
            }}>
            <Text
              style={{
                flex: 4,
                color: 'black',
                backgroundColor: '#888484',
                padding: 5,
                marginLeft: 20,
              }}>
              {isCaptureCCE?"CCE Parameters": "Capture CCE?"}
            </Text>
            <Switch
              trackColor={{false: '#767577', true: 'pink'}}
              thumbColor={isCaptureCCE ? 'violet' : '#f4f3f4'}
              ios_backgroundColor="#3e3e3e"
              onValueChange={() => {
                setIsCaptureCCE(t => {
                  dispatch(setIsCapturedCCE(!t))
                  return !t
                })
              }}
              value={isCaptureCCE}
              style={{
                marginRight: 20,
              }}
            />
          </View>
            {isCaptureCCE ? <CCE /> : null}
            <Description />

            {/* Photo Section */}
            <View
            style={{
              marginTop: 15,
              flexDirection: 'column',
              // marginHorizontal: 20,
            }}>
            <Text
              style={{
                color: 'black',
                backgroundColor: '#888484',
                padding: 5,
                marginHorizontal: 20,
              }}>
              Photo
            </Text>
          <FormCameraHandle />
          </View>
          <View
            style={{
              marginTop: 15,
              flexDirection: 'column',
              // marginHorizontal: 20,
            }}>
            <Text
              style={{
                color: 'black',
                backgroundColor: '#888484',
                padding: 5,
                marginHorizontal: 20,
              }}>
              Quality Control
            </Text>
          </View>
          <QualityControl navigation={navigation} scrollRef={scrollRef}></QualityControl>
        </View>
      </ScrollView>
    </>
  );
}
```

QualityControl.tsx

```
import {Alert, Button, Image, Text, View} from 'react-native';
import {saveToLocalStorage, storage} from '../../localStorage';
import store from '../../store';
import {useCallback, useEffect, useState} from 'react';
import {useDispatch, useSelector} from 'react-redux';
import {
  resetState,
  selectCapturedFromMap,
  selectDataCollection,
  selectIsCCECaptured,
  selectIsCCEGoingToBeCaptured,
  setLandCoverType,
  setLocationData,
} from '../../features/DataCollectionSlice';
// @ts-ignore
import CorrectIcon from './../../assets/correct-icon.png';
// @ts-ignore
import WrongIcon from './../../assets/wrong-icon.png';
import {
  reset,
  resetLocation,
  selectLocation,
} from '../../features/LocationSlice';
import {calculateExactLocation} from '../../location/getLocation';
function CorrectWrongIcon({
  isCorrect,
}: {
  isCorrect: boolean;
}): React.JSX.Element {
  return (
    <Image
      source={isCorrect ? CorrectIcon : WrongIcon}
      style={{
        height: 30,
        width: 30,
      }}
    />
  );
}
export default function ({
  navigation,
  scrollRef,
}: {
  navigation: any;
  scrollRef: any;
}) {
  const dataCollectionData = useSelector(selectDataCollection);
  const [isSavable, setIsSavable] = useState(false);
  const [photoCaptured, setPhotoCaptured] = useState(false);
  const [locationCaptured, setLocationCaptured] = useState(false);
  const [bearingToCenterCaptured, setBearingToCenterCaptured] = useState(false);
  const [distanceToCenterCaptured, setDistanceToCenterCaptured] =
    useState(false);
  const [landCoverTypeCaptured, setLandCoverTypeCaptured] = useState(false);
  const [cropInformationCaptured, setCropInformationCaptured] = useState(false);
  const CCECaptured = useSelector(selectIsCCECaptured);
  const capturedFromMap = useSelector(selectCapturedFromMap);
  const CCEGonnaBeCaptured = useSelector(selectIsCCEGoingToBeCaptured);
  // const [CCECaptured, setCCECaptured] = useState(false);
  const locationData = useSelector(selectLocation);
  const dispatch = useDispatch();

  useEffect(() => {
    // if capturing the CCE
    if (CCEGonnaBeCaptured) {
      if (dataCollectionData.landCoverType == 'Cropland') {
        // if it is a crop land
        setIsSavable(
          locationCaptured &&
            bearingToCenterCaptured &&
            distanceToCenterCaptured &&
            landCoverTypeCaptured &&
            CCECaptured &&
            cropInformationCaptured,
        );
      } else {
        setIsSavable(
          locationCaptured &&
            bearingToCenterCaptured &&
            distanceToCenterCaptured &&
            landCoverTypeCaptured &&
            CCECaptured,
        );
      }
    } else {
      // not capturing the CCE
      if (dataCollectionData.landCoverType == 'Cropland') {
        // if it is a crop land
        setIsSavable(
          locationCaptured &&
            bearingToCenterCaptured &&
            distanceToCenterCaptured &&
            landCoverTypeCaptured &&
            cropInformationCaptured,
        );
      } else {
        setIsSavable(
          locationCaptured &&
            bearingToCenterCaptured &&
            distanceToCenterCaptured &&
            landCoverTypeCaptured,
        );
      }
    }

    if (CCECaptured && capturedFromMap) {
      if (dataCollectionData.landCoverType == 'Cropland') {
        // if it is a crop land
        setIsSavable(
          locationCaptured &&
            landCoverTypeCaptured &&
            CCECaptured &&
            cropInformationCaptured,
        );
      } else {
        setIsSavable(locationCaptured && landCoverTypeCaptured && CCECaptured);
      }
    }
    if (CCECaptured == false && capturedFromMap) {
      if (dataCollectionData.landCoverType == 'Cropland') {
        // if it is a crop land
        setIsSavable(
          locationCaptured && landCoverTypeCaptured && cropInformationCaptured,
        );
      } else {
        setIsSavable(locationCaptured && landCoverTypeCaptured);
      }
    }
  }, [
    bearingToCenterCaptured,
    locationCaptured,
    capturedFromMap,
    distanceToCenterCaptured,
    landCoverTypeCaptured,
    cropInformationCaptured,
    CCECaptured,
    CCEGonnaBeCaptured,
    dataCollectionData,
  ]);

  useEffect(() => {
    dispatch(setLocationData(locationData));
  }, [locationData]);

  useEffect(() => {
    setLocationCaptured(
      dataCollectionData.latitude != null &&
        dataCollectionData.accuracyCorrection != null,
    );
    setDistanceToCenterCaptured(dataCollectionData.distanceToCenter != null);
    setBearingToCenterCaptured(dataCollectionData.bearingToCenter != null);
    setLandCoverTypeCaptured(dataCollectionData.landCoverType != null);

    if (dataCollectionData.landCoverType == 'Cropland') {
      let cropLandCondition =
        // dataCollectionData.cropInformation.isCaptured == true && // set this sometime
        dataCollectionData.cropInformation.waterSource != null &&
        dataCollectionData.cropInformation.cropIntensity != null &&
        dataCollectionData.cropInformation.primarySeason.cropName != null &&
        dataCollectionData.cropInformation.secondarySeason.cropName != null &&
        // dataCollectionData.cropInformation.liveStock != null &&
        dataCollectionData.cropInformation.croppingPattern != null;
      setCropInformationCaptured(cropLandCondition);
      console.log(cropLandCondition, 'crop');
    }

    setPhotoCaptured(dataCollectionData.images?.length >= 2);
  }, [dataCollectionData]);
  const saveToLocalStorageHandler = useCallback(() => {
    console.log(dataCollectionData.longitude, 'ooogitutde');
    const {latitude, longitude} = calculateExactLocation(
      dataCollectionData.latitude,
      dataCollectionData.longitude,
      dataCollectionData.distanceToCenter,
      dataCollectionData.bearingToCenter,
    );
    const finalDataCollectionData = {
      ...dataCollectionData,
      latitude: latitude,
      longitude: longitude,
    };
    saveToLocalStorage(finalDataCollectionData);
  }, [dataCollectionData]);

  return (
    <>
      <View
        style={{
          flexDirection: 'row',
          width: '100%',
          paddingHorizontal: 25,
          padding: 4,
          justifyContent: 'space-between',
          alignItems: 'center',
        }}>
        <Text
          style={{
            color: 'black',
          }}>
          Capture a photo of the area
        </Text>
        <CorrectWrongIcon isCorrect={photoCaptured} />
      </View>
      {capturedFromMap == false ? (
        <>
          <View
            style={{
              flexDirection: 'row',
              width: '100%',
              paddingHorizontal: 25,
              padding: 4,
              justifyContent: 'space-between',
              alignItems: 'center',
            }}>
            <Text
              style={{
                color: 'black',
              }}>
              Collect sufficient GPS points
            </Text>
            <CorrectWrongIcon isCorrect={locationCaptured} />
          </View>

          <View
            style={{
              flexDirection: 'row',
              width: '100%',
              paddingHorizontal: 25,
              padding: 4,
              justifyContent: 'space-between',
              alignItems: 'center',
            }}>
            <Text
              style={{
                color: 'black',
              }}>
              Captured bearing to center
            </Text>
            <CorrectWrongIcon isCorrect={bearingToCenterCaptured} />
          </View>

          <View
            style={{
              flexDirection: 'row',
              width: '100%',
              paddingHorizontal: 25,
              padding: 4,
              justifyContent: 'space-between',
              alignItems: 'center',
            }}>
            <Text
              style={{
                color: 'black',
              }}>
              Captured distance to center
            </Text>
            <CorrectWrongIcon isCorrect={distanceToCenterCaptured} />
          </View>
        </>
      ) : null}
      <View
        style={{
          flexDirection: 'row',
          width: '100%',
          paddingHorizontal: 25,
          padding: 4,
          justifyContent: 'space-between',
          alignItems: 'center',
        }}>
        <Text
          style={{
            color: 'black',
          }}>
          Land cover type captured
        </Text>
        <CorrectWrongIcon isCorrect={landCoverTypeCaptured} />
      </View>
      {dataCollectionData.landCoverType == 'Cropland' ? (
        <View
          style={{
            flexDirection: 'row',
            width: '100%',
            paddingHorizontal: 25,
            padding: 4,
            justifyContent: 'space-between',
            alignItems: 'center',
          }}>
          <Text
            style={{
              color: 'black',
            }}>
            Crop information captured
          </Text>
          <CorrectWrongIcon isCorrect={cropInformationCaptured} />
        </View>
      ) : null}
      {CCEGonnaBeCaptured == true ? (
        <View
          style={{
            flexDirection: 'row',
            width: '100%',
            paddingHorizontal: 25,
            padding: 4,
            justifyContent: 'space-between',
            alignItems: 'center',
          }}>
          <Text
            style={{
              color: 'black',
            }}>
            CCE captured
          </Text>
          <CorrectWrongIcon isCorrect={CCECaptured} />
        </View>
      ) : (
        false
      )}

      <View
        style={{
          marginHorizontal: 25,
          margin: 5,
        }}>
        <Button
          title="submit"
          onPress={() => {
            if (isSavable) {
              saveToLocalStorageHandler();
              scrollRef.current?.scrollTo({
                y: 0,
                animated: true,
              });
              dispatch(resetState());
              dispatch(reset());
              dispatch(resetLocation());
              navigation.goBack();
            } else {
              Alert.alert(
                'Make sure you have filled the entries properly',
                'Make sure to fill all the important fields before saving',
              );
            }
          }}></Button>
      </View>
      <View
        style={{
          marginHorizontal: 25,
          margin: 5,
        }}>
        <Button
          title="Cancel"
          onPress={() => {
            dispatch(resetState());
            console.log(storage.getAllKeys());
            scrollRef.current?.scrollTo({
              y: 0,
              animated: true,
            });
            navigation.goBack();
          }}></Button>
      </View>
    </>
  );
}

```

SeasonSelection.tsx

```
import {Text, View} from 'react-native';
import CustomModal from '../CustomModal2';
import {cropsData, seasonData} from '../../data';
import {useDispatch} from 'react-redux';
import {
  setPrimaryCrop,
  setPrimarySeason,
  setSecondaryCrop,
  setSecondarySeason,
} from '../../features/DataCollectionSlice';
import CustomModalWithTextBox from '../CustomModalWithTextBox';

export default function () {
  const dispatch = useDispatch();

  return (
    <>
      <View
        style={{
          flexDirection: 'row',
          alignItems: 'center',
          justifyContent: 'center',
          //   width:"100%",
          marginHorizontal: 25,
          marginTop: 5,
        }}>
        <Text
          style={{
            color: 'black',
            flex: 5,
            // padding:5
          }}>
          Season 1
        </Text>
        <CustomModal
          data={seasonData}
          action={payload => dispatch(setPrimarySeason(payload))}></CustomModal>
        <CustomModalWithTextBox
          data={cropsData}
          action={payload =>
            dispatch(setPrimaryCrop(payload))
          }></CustomModalWithTextBox>
        {/* <CustomModal
            data={cropsData}
            action={payload => dispatch(setPrimaryCrop(payload))}></CustomModal> */}
      </View>
      <View
        style={{
          flexDirection: 'row',
          alignItems: 'center',
          justifyContent: 'center',
          //   width:"100%",
          marginHorizontal: 25,
          marginTop: 5,
        }}>
        <Text
          style={{
            color: 'black',
            flex: 5,
            // padding:5
          }}>
          Season 2
        </Text>

        <CustomModal
          data={seasonData}
          action={payload => {
            dispatch(setSecondarySeason(payload));
          }}></CustomModal>
        <CustomModalWithTextBox
          data={cropsData}
          action={payload => {
            dispatch(setSecondaryCrop(payload));
          }}></CustomModalWithTextBox>
        {/* <CustomModal
            data={cropsData}
            action={payload =>{
              dispatch(setSecondaryCrop(payload))
              console.log(payload)
            }
            }></CustomModal> */}
      </View>
    </>
  );
}
```

- Now about the `/data` folder, has `index.ts` which has all the data for the form collection.

index.ts

```


export const waterSourceData = [
    {
        value: 1,
        title: "Unknown"
    },
    {
        value: 2,
        title: "Rainfed"
    },
    {
        value: 3,
        title: "Irrigated"
    },

]


export const landData = [
    {
        value: 1,
        title: "Unknown"
    },
    {
        value: 2,
        title: "Cropland"
    },
    {
        value: 3,
        title: "Forest"
    },
    {
        value: 4,
        title: "Grassland"
    },
    {
        value: 5,
        title: "Barren"
    },
    {
        value: 6,
        title: "Builtup"
    },
    {
        value: 7,
        title: "Shrub"
    },
]

export const cropIntensityData = [
    {
        value: 1,
        title: "Unknown"
    },
    {
        value: 2,
        title: "Single"
    },
    {
        value: 3,
        title: "Double"
    },
    {
        value: 4,
        title: "Triple"
    },
    {
        value: 5,
        title: "Continuous"
    },
]


export const cropsData = [
    {
        value: 1,
        title: "Unknown"
    },
    {
        value: 2,
        title: "PigeonPea"
    },
    {
        value: 3,
        title: "Chickpea"
    },
    {
        value: 4,
        title: "Wheat"
    },
    {
        value: 5,
        title: "Maize(Corn)"
    },
    {
        value: 6,
        title: "Rice"
    },
    {
        value: 7,
        title: "Barley"
    },
    {
        value: 8,
        title: "SoyaBean"
    },
    {
        value: 9,
        title: "Pulses"
    },
    {
        value: 10,
        title: "Cotton"
    },
    {
        value: 11,
        title: "Potatoe"
    },
    {
        value: 12,
        title: "Alfalfa"
    },
    {
        value: 13,
        title: "Sorghum"
    },
    {
        value: 14,
        title: "Millet"
    },
    {
        value: 15,
        title: "Sunflower"
    },
    {
        value: 16,
        title: "Rye"
    },
    {
        value: 17,
        title: "Rapeseed or Canola"
    },
    {
        value: 18,
        title: "Sugarcane"
    },
    {
        value: 19,
        title: "Groundnut or Peanut"
    },
    {
        value: 20,
        title: "Cassava"
    },
    {
        value: 21,
        title: "Sugarbeet"
    },
    {
        value: 22,
        title: "Palm"
    },
    {
        value: 23,
        title: "Others"
    },
    {
        value: 24,
        title: "Plantation"
    },
    {
        value: 25,
        title: "Fallow"
    },
    {
        value: 26,
        title: "Tef"
    },
]

export const livestockData = [
    {
        value: 1,
        title: "Unknown"
    },
    {
        value: 2,
        title: "Cows"
    },
    {
        value: 3,
        title: "Buffaloes"
    },
    {
        value: 4,
        title: "Goats"
    },
    {
        value: 5,
        title: "Sheep"
    },
]


export const croppingPatternData = [
    {
        value: 1,
        title: "Unknown"
    },
    {
        value: 2,
        title: "Continuous / Monocropping"
    },
    {
        value: 3,
        title: "Mixed"
    },
    {
        value: 4,
        title: "Intercropping"
    }
]


export const seasonData = [
    {
        value: 1,
        title: "Unknown"
    },
    {
        value: 2,
        title: "Kharif"
    },
    {
        value: 3,
        title: "Rabi"
    },
    {
        value: 4,
        title: "Summer/Dry/Zaid"
    },
    {
        value: 5,
        title: "Meher"
    },
    {
        value: 6,
        title: "Belg"
    },
    {
        value: 7,
        title: "Aus"
    },
]


export const cropGrowthStageData = [
    {
        value: 1,
        title: "Unknown"
    },
    {
        value: 2,
        title: 'Vegetative'
    },
    {
        value: 3,
        title: "Flowering"
    },
    {
        value: 4,
        title: "Harvesting"
    }
]
```

- As we're using `redux` to update the state, and do state-management. We create three slices and use them seperately at most places.

DataCollectionSlice.tsx

```
import {PayloadAction, createSlice} from '@reduxjs/toolkit';

const initialState = {
  // latitude: null,
  // longitude: null,
  capturedFromMaps: false,
  latitude: null,
  longitude: null,
  accuracyCorrection: null,
  bearingToCenter: null,
  distanceToCenter: null,
  landCoverType: null,
  cropInformation: {
    isCaptured: false,
    waterSource: null,
    cropIntensity: null,
    primarySeason: {
      seasonName: null,
      cropName: null,
    },
    secondarySeason: {
      seasonName: null,
      cropName: null,
    },
    // liveStock: null,
    croppingPattern: null,
    cropGrowthStage: null,
    remarks: null,
  },
  CCE: {
    isCaptured: false,
    isGoingToBeCaptured: false, // omit this in the server request
    sampleSize: {
      x: null, // change this to sampleSize1
      y: null, // change this to sampleSize2
    },
    grainWeight: null,
    biomassWeight: null,
    cultivar: null,
    sowDate: null,
    harvestDate: null,
  },
  images: [null], //null for typescript to infer a string type, so remove that null
  locationDesc: null,
};
export const counterSlice = createSlice({
  name: 'dataform',
  initialState: initialState,
  reducers: {
    setCapturedFromMap: (state, action) => {
      state.capturedFromMaps = action.payload;
      console.log(action);
    },
    setLocationData: function (state, action) {
      state.latitude = action.payload.latitude;
      state.longitude = action.payload.longitude;
      state.accuracyCorrection = action.payload.accuracy;
    },
    setBearingToCenterData: function (state, action) {
      state.bearingToCenter = action.payload;
    },
    setDistanceToCenterData: (state, action) => {
      state.distanceToCenter = action.payload;
    },

    setLandCoverType: (state, action) => {
      state.landCoverType = action.payload;
    },
    setWaterSource: (state, action) => {
      state.cropInformation.waterSource = action.payload;
    },
    setCropIntensity: (state, action) => {
      state.cropInformation.cropIntensity = action.payload;
    },
    setPrimarySeason: (state, action) => {
      state.cropInformation.primarySeason.seasonName = action.payload;
    },
    setSecondarySeason: (state, action) => {
      state.cropInformation.secondarySeason.seasonName = action.payload;
    },
    setPrimaryCrop: (state, action) => {
      console.log('check');
      state.cropInformation.primarySeason.cropName = action.payload;
    },
    setSecondaryCrop: (state, action) => {
      console.log(action);
      state.cropInformation.secondarySeason.cropName = action.payload;
    },
    // deleteSeason: (state, action) => {
    //   state.cropInformation.additionalSeasons.splice(action.payload, 1);
    // },
    // setLiveStock: (state, action) => {
    //   state.cropInformation.liveStock = action.payload;
    // },
    setCroppingPattern: (state, action) => {
      state.cropInformation.croppingPattern = action.payload;
    },
    setIsCapturedCCE: (state, action) => {
      state.CCE.isGoingToBeCaptured = action.payload;
    },
    setXSampleSize: (state, action) => {
      console.log(action.payload);
      state.CCE.sampleSize.x = action.payload;
    },
    setYSampleSize: (state, action) => {
      state.CCE.sampleSize.y = action.payload;
    },
    setGrainWeight: (state, action) => {
      state.CCE.grainWeight = action.payload;
    },
    setBiomassWeight: (state, action) => {
      state.CCE.biomassWeight = action.payload;
    },
    setLocationDescription: (state, action) => {
      state.locationDesc = action.payload;
    },
    setCropRemarks: (state, action) => {
      state.cropInformation.remarks = action.payload;
    },
    //images section
    addImage: (state, action) => {
      state.images.push(action.payload);
    },
    removeImage: (state, action) => {
      state.images = state.images.filter(value => {
        if (value == action.payload) return;
        return true;
      });
    },
    setCCECaptured: (state, action) => {
      console.log(action.payload);
      state.CCE.isCaptured = action.payload;
    },
    setCCESowDate: (state, action) => {
      console.log('yep');
      state.CCE.sowDate = action.payload;
    },
    setCCEHarvestDate: (state, action) => {
      state.CCE.harvestDate = action.payload;
    },
    setCultivar: (state, action) => {
      state.CCE.cultivar = action.payload;
    },
    setCropGrowthStage: (state, action) => {
      state.cropInformation.cropGrowthStage = action.payload;
    },
    resetState: state => initialState,
  },
});

export const {
  setLandCoverType,
  setCroppingPattern,
  setWaterSource,
  setCropIntensity,
  setPrimaryCrop,
  setSecondaryCrop,
  setBearingToCenterData,
  setDistanceToCenterData,
  setXSampleSize,
  setYSampleSize,
  // setLiveStock,
  addImage,
  setLocationData,
  removeImage,
  setPrimarySeason,
  setSecondarySeason,
  setIsCapturedCCE,
  setCCESowDate,
  setCCEHarvestDate,
  resetState,
  setCropGrowthStage,
  setGrainWeight,
  setCCECaptured,
  setCapturedFromMap,
  setBiomassWeight,
  setCropRemarks,
  setLocationDescription,
  setCultivar,
} = counterSlice.actions;

export default counterSlice.reducer;

export const selectCapturedFromMap = (state: any) =>
  state.dataform.capturedFromMaps;
export const selectLandCoverType = (state: any) => state.dataform.landCoverType;
export const selectWaterSourceType = (state: any) =>
  state.dataform.cropInformation.waterSource;
export const selectImagesList = (state: any) => state.dataform.images;
export const selectAdditionalSeasons = (state: any) =>
  state.dataform.cropInformation.additionalSeasons;
export const selectDataCollection = (state: any) => state.dataform;
export const selectCCEData = (state: any) => state.dataform.CCE;
export const selectPrimaryCrop = (state: any) =>
  state.dataform.cropInformation.primarySeason.cropName;
export const selectSecondaryCrop = (state: any) =>
  state.dataform.cropInformation.secondarySeason.cropName;
export const selectIsCCECaptured = (state: any) =>
  state.dataform.CCE.isCaptured;
export const selectIsCCEGoingToBeCaptured = (state: any) =>
  state.dataform.CCE.isGoingToBeCaptured;
export const selectBearingToCenter = (state: any) =>
  state.dataform.bearingToCenter;
export const selectDistanceToCenter = (state: any) =>
  state.dataform.distanceToCenter;
export const selectCropGrowthStage = (state: any) =>
  state.dataform.cropInformation.cropGrowthStage;
```

LocationSlice.tsx

```
import {createSlice} from '@reduxjs/toolkit';

const locationSlice = createSlice({
  name: 'location',
  initialState: {
    latitude: null,
    longitude: null,
    accuracy: null,
    degreesToNorth: 0,
    resetter: 0.9,
  },
  reducers: {
    setLatitude: (state, action) => {
      state.latitude = action.payload;
    },
    reset: state => {
      state.resetter = Math.random();
    },
    resetLocation: state => {
      state.longitude = null;
      state.latitude = null;
      state.accuracy = null;
    },
    setLongitude: (state, action) => {
      state.longitude = action.payload;
    },
    setLocation: (state, action) => {
      state.longitude = action.payload.longitude;
      state.latitude = action.payload.latitude;
      state.accuracy = action.payload.accuracy;
    },
    clearLocation: state => {
      state.longitude = null;
      state.latitude = null;
      state.accuracy = null;
    },
    setDegreesToNorth: (state, action) => {
      state.degreesToNorth = action.payload;
    },
  },
});

export const {
  setLatitude,
  setLongitude,
  clearLocation,
  setDegreesToNorth,
  resetLocation,
  setLocation,
  reset,
} = locationSlice.actions;
export default locationSlice.reducer;
export const selectLocation = (state: any) => state.location;
export const selectDegreesToNorth = (state: any) =>
  state.location.degreesToNorth;
export const selectResetter = (state: any) => state.location.resetter;
```

UISlice.tsx

```
import { createSlice } from "@reduxjs/toolkit";
import { retrieveAllData } from "../localStorage";
import { cropsData, landData, waterSourceData } from "../data";


const uiSlice = createSlice({
    name: "ui",
    initialState: {
        waterSource: [{}],
        cropType: [{}],
        landCover: [{}]
    },
    reducers: {
        setWaterSourceCropTypeLandCover: (state) => {
            const dataArray = retrieveAllData();
            // console.log("HELLo", dataArray)
            const w:any ={}
            const c:any = {}
            const l:any = {}
            const cropArr = []
            const waterArr = []
            const landArr = []
            for(const element of dataArray){
                const cropType = (element?.cropInformation?.primarySeason?.cropName)
                const waterType = (element?.cropInformation?.waterSource)
                const landType = (element?.landCoverType);

                if(!w[waterType]){
                    w[waterType] = 1;
                }
                else{
                    w[waterType]++;
                }
                if(!c[cropType]){
                    c[cropType] = 1;
                }
                else{
                    c[cropType]++;
                }
                if(!l[landType]){
                    l[landType] = 1;
                }
                else{
                    l[landType]++;
                }
                state.cropType = c;
                state.landCover = l;
                state.waterSource = w;
            }
            for(let a in l){
                if(a == "null" || a == null || typeof a == typeof {} || l[a] == undefined) continue;
                landArr.push({
                    title: a,
                    value: l[a]
                })
            }
            for(let a in c){
                if(a == "null" || a == null || typeof a == typeof {} || c[a] == undefined) continue;
                cropArr.push({
                    title: a,
                    value: c[a]
                })
            }
            for(let a in w){
                if(a == "null" || a == null || typeof a == typeof {} || w[a] == undefined) continue;
                waterArr.push({
                    title: a,
                    value: w[a]
                })
            }
            state.waterSource = waterArr
            state.cropType = cropArr
            state.landCover = landArr
            console.log(state)
        }

    }
})

export const selectUiData = (state: any) => state.ui
export const {setWaterSourceCropTypeLandCover} = uiSlice.actions
export default uiSlice.reducer
```

- The help page inside of `help` folder, this is used to show the help page through the drawer.

HelpPage.tsx

```
import React from 'react';
import {ScrollView, Text, View} from 'react-native';

const helpTexts = [
  'Collect Data on areas greater than 90 by 90 meter if you are unable to find a pure area of 90 by 90 meters and a second crop is present, please document it using the secondary crop field and season.',
  'If you are not at the center of the area. Capture the bearing and distance to the center of the field.',
  'Try to space your data collection by over 1 KM unless nearby samples units are of differetn classes.',
  'Two or three photos of each location with one covering the entire field and another with a close up of the crop. Please take your photos in landscape view from where you record the position.',
  'Cropland is defined as all cultivated plants harvested for food, feed, and fiber including plantations (e.g., orchards, vineyards, coffee, tea, rubber).',
  'Cropland intensity is the number of cropping cycles in a 12 month period.',
  'Irrigation is defined as artificial application of any amount of water to overcome crop water stress. Irrigated areas are those areas which are irrigated one or more times during the crop growing season.',
  'If the area has access to internet connection, you can use the mark location from maps feature, the location coordinates can be chosen from the map',
  "If there's a crop that isn't present in the list of crops, then the option of adding miscellaneous crop can be used",
];

function HelpPage() {
  return (
    <ScrollView>
      {helpTexts.map((value, id) => {
        return (
          <View
            key={id}
            style={{
              padding: 5,
              backgroundColor: '#d4d4d4',
            }}>
            <Text
              style={{
                fontSize: 18,
                margin: 2,
                padding: 7,
                color: 'black',
                borderBlockColor: 'grey',
                borderWidth: 0.5,
                backgroundColor: 'white',
              }}>
              {value}
            </Text>
          </View>
        );
      })}
    </ScrollView>
  );
}

export default HelpPage;

```

- Local storage, this has the code for mainting and setting data in the local storage of the device, which has to be synced later, and also for authorization purposes.

```
import {MMKV} from 'react-native-mmkv';

export const storage = new MMKV();

export function saveToLocalStorage(params: any) {

  if(!storage.contains('bottomIndex')){
    storage.set('bottomIndex', 1);
  }
  //counting the whole count of the data synced, it is updated to the latest
  if (!storage.contains('counter')) {
    storage.set('counter', 1);
  } else {
    let count = storage.getNumber('counter');
    if (count != undefined) {
      count++;
      storage.set('counter', count);
    }
  }



  // the format of storing the data is like this --> data.entry-(count)
  // it goes like this --> data.entry-1
  const finalParams = JSON.stringify(params);
  const count = storage.getNumber("counter");
  storage.set(`data.entry-${count}`, finalParams);

  //finally it is all set.
}

export function hell() {
  // storage.set('user.name', 'Marc')
  // storage.set('user.age', 21)
  // storage.set('is-mmkv-fast-asf', true)

  // const username = storage.getString('user.name') // 'Marc'
  // const age = storage.getNumber('user.age') // 21
  // const isMmkvFastAsf = storage.getBoolean('is-mmkv-fast-asf') // true

  // console.log(username, age, isMmkvFastAsf);

  // // checking if a specific key exists
  // const hasUsername = storage.contains('user.name')

  // getting all keys
  const keys = storage.getAllKeys(); // ['user.name', 'user.age', 'is-mmkv-fast-asf']

  // delete a specific key + value
  storage.delete('user.name');

  // delete all keys
  // storage.clearAll()

  console.log(keys);

  // const user = {
  //     username: 'Marc',
  //     age: 21
  // }

  // // Serialize the object into a JSON string
  // storage.set('user', JSON.stringify(user))

  // // Deserialize the JSON string into an object
  // const jsonUser = storage.getString('user') // { 'username': 'Marc', 'age': 21 }
  // // @ts-ignore
  // const userObject = JSON.parse(jsonUser)
  // console.log(userObject)
}


export function retrieveAllData(){
    let a = storage.getNumber('bottomIndex')
    let totalCount = storage.getNumber('counter')
    const dataObject: any[] = [];
    if(a && totalCount){
      for(; a <= totalCount; a++){
          let dataString = storage.getString(`data.entry-${a}`);
          if(dataString){
            let data = JSON.parse(dataString);
            let newData = {
              ...data,

            }
            newData.CCE.sampleSize1 = data.CCE.sampleSize.x;
            newData.CCE.sampleSize2 = data.CCE.sampleSize.y;
            newData.CCE.sampleSize = null;
            let newImageList = data.images.filter((value:any) => (value != null))
            newData.images = newImageList;
            dataObject.push(newData);
          }
      }
    }
    return dataObject;
}


export function setBottomIndex(bottomIndex: number){
  // while a data entry is synced successfully, we need not change the counter, the counter is for the top one, and hence just the bottom needs to be changed.
  storage.delete(`data.entry-${bottomIndex-1}`)
  storage.set('bottomIndex', bottomIndex);
  // storage.set('count', count);
  return true;
}
export function getBottomIndexCount(){
  return storage.getNumber('bottomIndex');
}

export function setJwtEmail(jwt: string, email: string){
  storage.set('user.email', email)
  storage.set('user.jwt', jwt)
}

export function getJwt() {
  return storage.getString("user.jwt");
}
```

- `location`, this contains functions which return the current location, in the image, and in the map data collection.

```
import Geolocation from '@react-native-community/geolocation';
import {useEffect, useState} from 'react';
import {Alert, Button, Text} from 'react-native';
import {useDispatch, useSelector} from 'react-redux';
import {selectLocation, setLatitude, setLocation} from '../features/LocationSlice';
// import {setLongitude} from '../features/DataCollectionSlice';

// Geolocation.getCurrentPosition(info => console.log(info));

// this is the structure of the value returned by this API
const something = {
  coords: {
    accuracy: 3.0999999046325684,
    altitude: 193.9,
    heading: 300.6000061035156,
    latitude: 17.983909999999998,
    longitude: 79.53571333333333,
    speed: 1.4607816934585571,
  },
  extras: {maxCn0: 41, meanCn0: 31, satellites: 21},
  mocked: false,
  timestamp: 1710641025000,
};

export default function () {
  const dispatch = useDispatch();
  const getCurrentPosition = () => {
    Geolocation.getCurrentPosition(
      (pos: any) => {
        // setPosition(JSON.stringify(pos));
        dispatch(setLocation(pos.coords))
        // dispatch(setLatitude(pos.coords.latitude));
        // dispatch(setLongitude(pos.coords.longitude));
        // console.log(locationData);
        console.log(pos)
      },
      (error: any) =>
        Alert.alert('GetCurrentPosition Error', JSON.stringify(error)),
      {enableHighAccuracy: true},
    );
  };
  getCurrentPosition();
}

export function getImageLocation() {
  let position = null;
  Geolocation.getCurrentPosition(
    (pos: any) => {
      // setPosition(JSON.stringify(pos));
      // dispatch(setLatitude(pos.coords.latitude));
      // dispatch(setLongitude(pos.coords.longitude));
      // console.log(locationData);
      position = pos;
      console.log(pos)
    },
    (error: any) =>
      Alert.alert('GetCurrentPosition Error', JSON.stringify(error)),
    {enableHighAccuracy: true},
  );
  return position;
}


export function calculateExactLocation(lat: number, lon: number, distance: number, bearing: number): { latitude: number, longitude: number } {
  const dist: number = distance / 6371000.0;
  const brng: number = (bearing * Math.PI) / 180;
  const lat1: number = (lat * Math.PI) / 180;
  const lon1: number = (lon * Math.PI) / 180;

  let exactLat: number = Math.asin(Math.sin(lat1) * Math.cos(dist) +
      Math.cos(lat1) * Math.sin(dist) * Math.cos(brng));

  let a: number;
  let denominator: number = Math.cos(dist) - Math.sin(lat1) * Math.sin(exactLat);
  if (Math.abs(denominator) < Number.EPSILON) {
      a = 0;
  } else {
      a = Math.atan2(Math.sin(brng) * Math.sin(dist) * Math.cos(lat1), denominator);
  }

  let exactLon: number = lon1 + a;
  exactLon = ((exactLon + 3 * Math.PI) % (2 * Math.PI)) - Math.PI;

  // Convert back to degrees
  exactLat = (exactLat * 180) / Math.PI;
  exactLon = (exactLon * 180) / Math.PI;

  return { latitude: exactLat, longitude: exactLon };
}
```

- `maps`
