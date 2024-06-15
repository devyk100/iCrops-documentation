# iCrops dashboard frontend

### This is the frontend of the iCrops dashboard, built using standard React, Redux, and Tailwind CSS. It also uses Typescript.

The dashboard is designed to provide an intuitive interface for users to visualize and interact with their crop data.

### The other dependencies used by this projects can be seen in the `package.json` file below:

```
{
  "name": "icrops-frontend",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "lint": "eslint . --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
    "preview": "vite preview"
  },
  "dependencies": {
    "@reduxjs/toolkit": "^2.2.3",
    "@types/uuid": "^9.0.8",
    "axios": "^1.6.8",
    "dotenv": "^16.4.5",
    "js-file-download": "^0.4.12",
    "localforage": "^1.10.0",
    "match-sorter": "^6.3.4",
    "nanoid": "^5.0.7",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-redux": "^9.1.0",
    "react-router-dom": "^6.22.3",
    "redux": "^5.0.1",
    "sort-by": "^1.2.0",
    "uuid": "^10.0.0"
  },
  "devDependencies": {
    "@types/react": "^18.2.66",
    "@types/react-dom": "^18.2.22",
    "@typescript-eslint/eslint-plugin": "^7.2.0",
    "@typescript-eslint/parser": "^7.2.0",
    "@vitejs/plugin-react": "^4.2.1",
    "autoprefixer": "^10.4.19",
    "eslint": "^8.57.0",
    "eslint-plugin-react-hooks": "^4.6.0",
    "eslint-plugin-react-refresh": "^0.4.6",
    "postcss": "^8.4.38",
    "tailwindcss": "^3.4.3",
    "typescript": "^5.4.5",
    "vite": "^5.2.0"
  }
}
```

# The project structure

```
- public // has all the photos and images
- src // the main folder where source code lies
    - assets // contains some of the assets like images, etc.,
    - features // redux slices
        data.ts
        ui.ts
    - pages
        - components
            - Modal
                ClickableDropdown.tsx
                DownloadDataWithImages.tsx
                GenerateDataFile.tsx
                Modal.tsx
            Button.tsx
            CCEInformationData.tsx
            CheckBox.tsx
            CropInformationData.tsx
            CheckBox.tsx
            CropInformationData.tsx
            Downloads.tsx
            Footer.tsx
            ImagesDownload.tsx
            Navbar.tsx
            TableHeadComponent.tsx
            TableRowComponent.tsx
        - dashboard
            Dashboard.tsx
            dataList.ts
            TableHead.tsx
        ImageFromFilename.tsx
        Landing.tsx
        Login.tsx
        SignUp.tsx
        SpecificData.tsx
        Spinner.tsx
    - store // the redux store
        index.ts
    App.css
    App.tsx
    axios_defaults.ts
    index.ts
    main.tsx
    vite-env.d.ts
    index.html
    package.json
    README.md
    tailwind.config.js
    tsconfig.json
    tsconfig.node.json
    vite.config.ts
```

To run this in dev mode - which has fast refresh to see changes quick, run `npm run dev` and visit `localhost:5173 ` or whatever port it has opened.

To make the production build, you should run `npm run build` and it will create a build for you, but make sure you follow ESLint rules. The build will be found in the `dist/` folder which can be hosted statically, as its just HTML, CSS, and JavaScript.

# Understanding the code.

Let us start with the `features` folder inside of
the `src` folder.

data.ts, a basic data redux slice and reducers for data

```
import {  createSlice } from "@reduxjs/toolkit";
// import { BACKEND_URL } from "../App";
// import axios from "axios";

export const dataSlice = createSlice({
    name: 'data',
    initialState: {
      overallData: null,
      filterData: {
        latitude: null,
        longitude: null,
        accuracy: null,
        landCover: null,
        description: null,
        email: null,
        sampleSize_1: null,
        sampleSize_2: null,
        biomassWeight: null,
        cultivar: null,
        sowDate: null,
        harvestDate: null,
        waterSource: null,
        cropIntensity: null,
        primarySeason: null,
        primaryCrop: null,
        secondarySeason: null,
        secondaryCrop: null,
        livestock: null,
        croppingPattern: null,
        cropGrowthStage: null,
        remarks: null,
        pageNo: 1,
        entries: 10,
        count: 0
      },
    },
    reducers: {
      setLatitude: (state, action) => {
        state.filterData.latitude = action.payload;
      },
      setLongitude: (state, action) => {
        state.filterData.longitude = action.payload;
      },
      setAccuracy: (state, action) => {
        state.filterData.accuracy = action.payload;
      },
      setLandCover: (state, action) => {
        console.log("payload")
        state.filterData.landCover = action.payload;
      },
      setDescription: (state, action) => {
        state.filterData.description = action.payload;
      },
      setEmail: (state, action) => {
        state.filterData.email = action.payload;
      },
      setSampleSize1: (state, action) => {
        state.filterData.sampleSize_1 = action.payload;
      },
      setSampleSize2: (state, action) => {
        state.filterData.sampleSize_2 = action.payload;
      },
      setBiomassWeight: (state, action) => {
        state.filterData.biomassWeight = action.payload;
      },
      setCultivar: (state, action) => {
        state.filterData.cultivar = action.payload;
      },
      setSowDate: (state, action) => {
        state.filterData.sowDate = action.payload;
      },
      setHarvestDate: (state, action) => {
        state.filterData.harvestDate = action.payload;
      },
      setWaterSource: (state, action) => {
        state.filterData.waterSource = action.payload;
      },
      setCropIntensity: (state, action) => {
        state.filterData.cropIntensity = action.payload;
      },
      setPrimarySeason: (state, action) => {
        state.filterData.primarySeason = action.payload;
      },
      setPrimaryCrop: (state, action) => {
        state.filterData.primaryCrop = action.payload;
      },
      setSecondarySeason: (state, action) => {
        state.filterData.secondarySeason = action.payload;
      },
      setSecondaryCrop: (state, action) => {
        state.filterData.secondaryCrop = action.payload;
      },
      setLivestock: (state, action) => {
        state.filterData.livestock = action.payload;
      },
      setCroppingPattern: (state, action) => {
        state.filterData.croppingPattern = action.payload;
      },
      setCropGrowthStage: (state, action) => {
        state.filterData.cropGrowthStage = action.payload;
      },
      setRemarks: (state, action) => {
        state.filterData.remarks = action.payload;
      }
      ,
      movePageForward: (state) => {
        console.log(state.filterData.count, "is the count")
        if(state.filterData.pageNo * state.filterData.entries < state.filterData.count) {
            state.filterData.pageNo += 1;
        }
        else{
            alert("Reached the end")
        }
      },
      movePageBack: (state) => {
        if(state.filterData.pageNo > 1){
            state.filterData.pageNo -= 1
        }
      },
      setCount: (state, action) => {
        state.filterData.count = action.payload
      },
      setEntries: (state, action) => {
        state.filterData.entries = action.payload
      }
    },
  });


  // Export action creators
  export const {
    setLatitude,
    setLongitude,
    setAccuracy,
    setLandCover,
    setDescription,
    setEmail,
    setSampleSize1,
    setSampleSize2,
    setBiomassWeight,
    setCultivar,
    setSowDate,
    setHarvestDate,
    setWaterSource,
    setCropIntensity,
    setPrimarySeason,
    setPrimaryCrop,
    setSecondarySeason,
    setSecondaryCrop,
    setLivestock,
    setCroppingPattern,
    setCropGrowthStage,
    setRemarks,
    setCount,
    movePageBack,
    movePageForward,
    setEntries
  } = dataSlice.actions;

export default dataSlice.reducer;
export const selectData = (state: any) => state.data.overallData;
export const selectFilterData = (state: any) => state.data.filterData;
export const selectPageNo = (state:any) => state.data.filterData.pageNo
export const selectEntries = (state:any) => state.data.filterData.entries
```

similarly, ui.ts

```
import { createSlice } from "@reduxjs/toolkit";

export const uiSlice = createSlice({
    name: "ui",
    initialState: {
        deletionOn: false,
        deletionList: [-1],
        deleteAll: false,
        columns: ["Latitude", "Longitude", "By", "Land Cover Type", "Primary Crop", "Secondary Crop"]
    },
    reducers: {
        deletionOn: (state) => {
            state.deletionOn = true
        },
        deletionOff: (state) => {
            state.deletionOn = false
        },
        addDeletionListElement: (state, action) => {
            if(action.payload != -1) state.deletionList.push(action.payload);
            console.log(state.deletionList)
        },
        removeDeletionListElement: (state, action) => {
            let duplicateList = state.deletionList.filter((value) => {
                if(value == action.payload) return false
                return true;
            })
            state.deletionList = duplicateList;
        },
        emptyList: (state) => {
            state.deletionList = [];
        },
        setDeleteAll: (state, action) => {
            state.deleteAll = action.payload;
        },
        addColumns: (state, action) => {
            let something = JSON.parse(JSON.stringify(state.columns))
            console.log(something)
            state.columns.push(action.payload);
        },
        removeColumns: (state, action) => {
            state.columns = state.columns.filter((value) => {
                if(value == action.payload) return false;
                return true;
            })
        }
    }
})

export const {deletionOff, deletionOn, addDeletionListElement, removeDeletionListElement, removeColumns, setDeleteAll, emptyList, addColumns} = uiSlice.actions
export default uiSlice.reducer
export const selectDeletion = (state:any) => state.ui.deletionOn
export const selectDeletionList = (state:any) => state.ui.deletionList
export const selectDeleteAll = (state:any) => state.ui.deleteAll
export const selectColumns = (state:any) => state.ui.columns
```

The `store` folder, which creates the final redux store.

```
import { configureStore } from '@reduxjs/toolkit'
import dataReducer from '../features/data'
import uiReducer from '../features/ui'

export default configureStore({
  reducer: {
    data: dataReducer,
    ui: uiReducer
  }
})

```

Top level `main.tsx` is the main entry point of this app

```
// import React from 'react'
import ReactDOM from "react-dom/client";
import App from "./App.tsx";
import "./index.css";
import { backendUrl } from "./axios_defaults.ts";
("./axios_defaults.ts");
console.log(backendUrl);
ReactDOM.createRoot(document.getElementById("root")!).render(<App />);
```

Which starts the flow with App.tsx which is as follows:

```
// import { useState } from 'react'
import "./App.css";
import {
  createBrowserRouter,
  isRouteErrorResponse,
  RouterProvider,
  useRouteError,
} from "react-router-dom";

export const BACKEND_URL = "http://maps.icrisat.org/";
// export const BACKEND_URL = "http://localhost:8090/";
export const startRoute = "/react";
const router = createBrowserRouter([
  {
    path: startRoute + "/data",
    element: <Dashboard />,
    // ErrorBoundary: ErrorBoundary
  },
  {
    path: startRoute + "/data/:dataid",
    element: <SpecificData />,
  },
  {
    path: startRoute + "/",
    element: <Landing />,
    ErrorBoundary: ErrorBoundary,
  },
  {
    path: startRoute + "/login",
    element: <Login />,
  },
  {
    path: startRoute + "/signup",
    element: <SignUp />,
  },
]);

function ErrorBoundary() {
  const error = useRouteError();
  if (isRouteErrorResponse(error)) {
    return (
      <div>
        <h1>Oops!</h1>
        <h2>{error.status}</h2>
        <p>{error.statusText}</p>
        {error.data?.message && <p>{error.data.message}</p>}
      </div>
    );
  } else {
    return <div>Oops some error occured in the application</div>;
  }
}

import Dashboard from "./pages/dashboard/Dashboard";
import SpecificData from "./pages/SpecificData";
import Landing from "./pages/Landing";
import Login from "./pages/Login";
import SignUp from "./pages/SignUp";
import store from "./store";
import { Provider } from "react-redux";
function App() {
  // const [count, setCount] = useState(0)
  return (
    <>
      <Provider store={store}>
        <RouterProvider router={router}></RouterProvider>
      </Provider>
      ,
    </>
  );
}
export default App;
```

It contains the main routing for the client, and also the backend based URL.

There are also other css files at the root level like
index.css which contains tailwind directives

```
@tailwind base;
@tailwind components;
@tailwind utilities;
```
