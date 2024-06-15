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

the uppermost root level HTML file is not touched often, but to change title and favicon, you can change it.

The `Modals` folder inside of `pages/components` is used for all kinds of modals for file, archive generation or column selection.

ClickableDropdown.tsx

```
import { MouseEventHandler, useState } from "react";
import Modal from "./Modal";
import Button from "../Button";
import { useDispatch, useSelector } from "react-redux";
import { addColumns, removeColumns, selectColumns } from "../../../features/ui";

const columnsList = [
  {
    name: "Latitude",
    value: "latitude",
  },
  {
    name: "Longitude",
    value: "longitude",
  },
  {
    name: "Accuracy",
    value: "accuracy",
  },
  {
    name: "Land Cover Type",
    value: "landCoverType",
  },
  {
    name: "Description",
    value: "description",
  },
  {
    name: "By",
    value: "by",
  },
  {
    name: "Water Source",
    value: "waterSource",
  },
  {
    name: "Crop Intensity",
    value: "cropIntensity",
  },
  {
    name: "Crop Growth Stage",
    value: "cropGrowthStage",
  },
  {
    name: "Cropping Pattern",
    value: "croppingPattern",
  },
  // {
  //     name: "Livestock",
  //     value: "livestock"
  // },
  {
    name: "Primary Crop",
    value: "primaryCrop",
  },
  {
    name: "Primary Season",
    value: "primarySeason",
  },
  {
    name: "Secondary Crop",
    value: "secondaryCrop",
  },
  {
    name: "Secondary Season",
    value: "secondarySeason",
  },
  {
    name: "Remarks",
    value: "remarks",
  },
  {
    name: "Sample Size 1",
    value: "sampleSize1",
  },
  {
    name: "Sample Size 2",
    value: "sampleSize2",
  },
  {
    name: "Biomass Weight",
    value: "biomassWeight",
  },
  {
    name: "Cultivar",
    value: "cultivar",
  },
  {
    name: "Sow Date",
    value: "sowDate",
  },
  {
    name: "Harvest Date",
    value: "harvestDate",
  },
];

function CheckBox({
  text,
  checked,
  onClick,
}: {
  text: string;
  key: string;
  checked?: boolean;
  onClick?: MouseEventHandler<HTMLInputElement>;
}) {
  let num = Math.floor(Math.random() * 100000);
  return (
    <div className="w-fit flex items-center transition hover:scale-110 my-1">
      {/* <input type="checkbox" className='mx-2 border-2 checked:border-red-500   accent-lime-400' id={`${num}`} /> */}
      <input
        type="checkbox"
        onClick={onClick}
        onChange={(event) => console.log(event.target.value)}
        checked={checked}
        className=" checked:bg-red-500 checked:border-transparent checked:ring-1 checked:ring-green-600 h-5 w-5 rounded-2xl accent-lime-300"
        id={`${num}`}
      />

      <label htmlFor={`${num}`} className="px-2 text-lg">
        {text}
      </label>
    </div>
  );
}

function ClickableDropdown({ closeHandler }: { closeHandler: () => void }) {
  const dispatch = useDispatch();
  const columns = useSelector(selectColumns);
  return (
    <Modal closeHandler={closeHandler}>
      <div className="bg-white p-10 rounded-lg bg-opacity-80">
        <div className="text-2xl font-bold mb-2">Select Columns to see</div>
        <form action="">
          {columnsList.map((value) => {
            let checked = false;
            for (let a of columns) {
              if (a == value.name) checked = true;
            }
            const [isChecked, setIsChecked] = useState(checked);
            return (
              <CheckBox
                onClick={() =>
                  setIsChecked((t) => {
                    if (t == false) {
                      dispatch(addColumns(value.name));
                    }
                    if (t == true) {
                      dispatch(removeColumns(value.name));
                    }
                    return !t;
                  })
                }
                text={value.name}
                checked={isChecked}
                key={value.value}
              ></CheckBox>
            );
          })}
        </form>
        <div className="mt-4">
          {/* <Button className='' onClick={() => close}>Confirm</Button> */}
          <Button className="" onClick={() => closeHandler()}>
            Close
          </Button>
        </div>
      </div>
    </Modal>
  );
}

export default ClickableDropdown;
```

DownloadDataWithImages.tsx

```
import { ReactNode, useCallback, useState } from "react";
import Modal from "./Modal";
import Button from "../Button";
import { BACKEND_URL } from "../../../App";
import axios from "axios";
import { useSelector } from "react-redux";
import { selectFilterData } from "../../../features/data";
import fileDownload from "js-file-download";
import { nanoid } from "nanoid";

enum fileType {
  csv,
  xlsx,
  shp,
}

function DownloadDataWithImages({
  closeHandler,
}: {
  closeHandler: () => void;
}) {
  const [buttonString, setButtonString] = useState<ReactNode>("Generate");
  const [isDisabled, setIsDisabled] = useState(false);
  const [archiveFormat, setArchiveFormat] = useState<fileType>(fileType.csv);
  const filterData = useSelector(selectFilterData);
  const generateArchiveRequest = useCallback(async () => {
    async function generator() {
      let finalURL = "";
      if (archiveFormat == fileType.csv) {
        finalURL = `${BACKEND_URL}api/v1/archive-data`;
      } else if (archiveFormat == fileType.xlsx) {
        finalURL = `${BACKEND_URL}api/v1/archive-data/xlsx`;
      }
      const response = await axios.post(
        finalURL,
        {
          pageNo: filterData.pageNo,
          entries: filterData.entries,
          latitude: filterData.latitude,
          longitude: filterData.longitude,
          accuracy: filterData.accuracy,
          landCover: filterData.landCover,
          description: filterData.description,
          email: filterData.email,
          sampleSize_1: filterData.sampleSize_1,
          sampleSize_2: filterData.sampleSize_2,
          biomassWeight: filterData.biomassWeight,
          cultivar: filterData.cultivar,
          sowDate: filterData.sowDate,
          harvestDate: filterData.harvestDate,
          waterSource: filterData.waterSource,
          cropIntensity: filterData.cropIntensity,
          primarySeason: filterData.primarySeason,
          primaryCrop: filterData.primaryCrop,
          secondarySeason: filterData.secondarySeason,
          secondaryCrop: filterData.secondaryCrop,
          // livestock: filterData.livestock,
          croppingPattern: filterData.croppingPattern,
          cropGrowthStage: filterData.cropGrowthStage,
          remarks: filterData.remarks,
        },
        {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
          responseType: "blob",
        }
      );
      console.log(response);
      await fileDownload(response.data, nanoid() + ".zip");
      setButtonString("Generate");
      setIsDisabled(false);
      // if (fileformat == filfeType.csv)
      // else if (fileformat == fileType.xlsx)
      //   fileDownload(response.data, nanoid()() + ".xlsx");
    }
    generator();
  }, [filterData, archiveFormat]);

  const radioClassName = "accent-red-500 w-5 h-5";
  const spanClassName = "py-1 w-fit gap-2 flex items-center text-xl";
  return (
    <Modal closeHandler={closeHandler}>
      <div className="bg-white bg-opacity-75 p-4 rounded-lg">
        <div className="text-xl font-semibold">Select the type of data</div>
        <form action="" className="flex flex-col">
          <span className={spanClassName}>
            <input
              defaultChecked
              type="radio"
              name="some"
              id="csvGDF"
              className={radioClassName}
              onClick={() => {
                setArchiveFormat(fileType.csv);
              }}
            />
            <label htmlFor="csvGDF">CSV</label>
          </span>
          <span className={spanClassName}>
            <input
              type="radio"
              name="some"
              id="xlsxGDF"
              className={radioClassName}
              onClick={() => {
                setArchiveFormat(fileType.xlsx);
              }}
            />
            <label htmlFor="xlsxGDF">XLSX</label>
          </span>
          <span>
            <Button
              disabled={isDisabled}
              onClick={async (event) => {
                setButtonString("Generating..");
                setIsDisabled(true);
                event.preventDefault();
                // functionalities
                await generateArchiveRequest();
              }}
              className="mt-2"
            >
              {buttonString}
            </Button>
            <Button
              onClick={(event) => {
                event.preventDefault();
                closeHandler();
              }}
            >
              Close
            </Button>
          </span>
        </form>
      </div>
    </Modal>
  );
}

export default DownloadDataWithImages;
```

GenerateDataFile.tsx

```
import { ReactNode, useCallback, useState } from "react";
import Modal from "./Modal";
import Button from "../Button";
import { BACKEND_URL } from "../../../App";
import { selectFilterData } from "../../../features/data";
import { useSelector } from "react-redux";
import axios from "axios";
import fileDownload from "js-file-download";
import { nanoid } from "nanoid";

enum fileType {
  csv,
  xlsx,
  shp,
}

function GenerateDataFile({ closeHandler }: { closeHandler: () => void }) {
  const filterData = useSelector(selectFilterData);
  const [fileformat, setFileformat] = useState<fileType>(fileType.csv);
  const radioClassName = "accent-red-500 w-5 h-5";
  const spanClassName = "py-1 w-fit gap-2 flex items-center text-xl";
  const [buttonString, setButtonString] = useState<ReactNode>("Generate");
  const [isDisabled, setIsDisabled] = useState(false);
  const generateFileRequest = useCallback(async () => {
    async function generator() {
      let finalURL = "";
      if (fileformat == fileType.csv) {
        finalURL = `${BACKEND_URL}api/v1/file-data`;
      } else if (fileformat == fileType.xlsx) {
        finalURL = `${BACKEND_URL}api/v1/file-data/xlsx`;
      } else if (fileformat == fileType.shp) {
        finalURL = `${BACKEND_URL}api/v1/file-data/shp`;
      }
      const response = await axios.post(
        finalURL,
        {
          pageNo: filterData.pageNo,
          entries: filterData.entries,
          latitude: filterData.latitude,
          longitude: filterData.longitude,
          accuracy: filterData.accuracy,
          landCover: filterData.landCover,
          description: filterData.description,
          email: filterData.email,
          sampleSize_1: filterData.sampleSize_1,
          sampleSize_2: filterData.sampleSize_2,
          biomassWeight: filterData.biomassWeight,
          cultivar: filterData.cultivar,
          sowDate: filterData.sowDate,
          harvestDate: filterData.harvestDate,
          waterSource: filterData.waterSource,
          cropIntensity: filterData.cropIntensity,
          primarySeason: filterData.primarySeason,
          primaryCrop: filterData.primaryCrop,
          secondarySeason: filterData.secondarySeason,
          secondaryCrop: filterData.secondaryCrop,
          // livestock: filterData.livestock,
          croppingPattern: filterData.croppingPattern,
          cropGrowthStage: filterData.cropGrowthStage,
          remarks: filterData.remarks,
        },
        {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
          responseType: "blob",
        }
      );
      console.log(response);
      if (fileformat == fileType.csv)
        await fileDownload(response.data, nanoid() + ".csv");
      else if (fileformat == fileType.xlsx)
        await fileDownload(response.data, nanoid() + ".xlsx");
      else if (fileformat == fileType.shp)
        await fileDownload(response.data, nanoid() + ".zip");
      setButtonString("Generate");
      setIsDisabled(false);
    }
    generator();
  }, [filterData, fileformat]);

  return (
    <Modal closeHandler={closeHandler}>
      <div className="bg-white bg-opacity-75 p-4 rounded-lg">
        <div className="text-xl font-semibold">Select the type of data</div>
        <form action="" className="flex flex-col">
          <span className={spanClassName}>
            <input
              type="radio"
              name="some"
              id="csvGDF"
              className={radioClassName}
              defaultChecked
              onClick={() => {
                setFileformat(fileType.csv);
              }}
            />
            <label htmlFor="csvGDF">CSV</label>
          </span>
          <span className={spanClassName}>
            <input
              type="radio"
              name="some"
              id="xlsxGDF"
              className={radioClassName}
              onClick={() => {
                setFileformat(fileType.xlsx);
              }}
            />
            <label htmlFor="xlsxGDF">XLSX</label>
          </span>
          <span className={spanClassName}>
            <input
              type="radio"
              name="some"
              id="shpGDF"
              className={radioClassName}
              onClick={() => {
                setFileformat(fileType.shp);
              }}
            />
            <label htmlFor="shpGDF">SHP</label>
          </span>
          <span>
            <Button
              disabled={isDisabled}
              onClick={async (event) => {
                setButtonString("Generating..");
                setIsDisabled(true);
                event.preventDefault();
                await generateFileRequest();
              }}
              className="mt-2"
            >
              {buttonString}
            </Button>
            <Button
              onClick={(event) => {
                event.preventDefault();
                closeHandler();
              }}
            >
              Close
            </Button>
          </span>
        </form>
      </div>
    </Modal>
  );
}

export default GenerateDataFile;
```

Modal.tsx // the basic handling of any modal,clsing and opening

```
import  { ReactNode } from 'react'

function Modal({ children, closeHandler }: {
  children: ReactNode;
  closeHandler?: () => void
}) {
  return (
    <div onClick={() => {
      if (closeHandler) closeHandler();
    }} className='flex y-5 items-center overflow-scroll justify-center h-screen w-screen absolute top-0 left-0 z-50 bg-gray-400 rounded-md bg-clip-padding backdrop-filter backdrop-blur-sm bg-opacity-10 border border-gray-100'>
      <div className='h-screen py-10' onClick={(event) => event.stopPropagation()}>
        {children}
      </div>
    </div>
  )
}
export default Modal
```

The `components` folder inside of `pages`

Button.tsx // consistent button ui

```
import  { MouseEventHandler, ReactNode } from 'react'

function Button({ children, onClick, className, disabled }: {
    children: ReactNode,
    onClick?: MouseEventHandler<HTMLButtonElement>,
    className?: string;
    disabled?: boolean;
}) {
    if(!disabled)
    return (
        <button onClick={onClick} className={'w-fit px-5 border border-gray-500  py-2 rounded-3xl font-semibold mx-1 transition hover:text-gray-100 ease-in-out delay-150 bg-blue-500 focus:-translate-y-1 hover:scale-110 hover:bg-indigo-500  '+className} disabled={disabled}>{children}</button>
    )
    return (
        <button onClick={onClick} className={'w-fit px-5 py-2 border border-gray-500  rounded-3xl font-semibold mx-1  bg-blue-500  '+className} disabled={disabled}>{children}</button>
    )
}

export default Button
```

CCEInformationData.tsx // code to show CCE info in the table

```
import { CCEType } from "../SpecificData"

export default function({CCEData}: {
    CCEData: CCEType
}){
    return (
        <>

        <td className="border-2 pr-2 border-t-0">{CCEData? CCEData.sampleSize_1: null}</td>
        <td className="border-2 pr-2 border-t-0">{CCEData? CCEData.sampleSize_2: null}</td>
        <td className="border-2 pr-2 border-t-0">{CCEData? CCEData.biomassWeight: null}</td>
        <td className="border-2 pr-2 border-t-0">{CCEData? CCEData.cultivar: null}</td>
        <td className="border-2 pr-2 border-t-0">{CCEData? CCEData.sowDate: null}</td>
        <td className="border-2 pr-2 border-t-0">{CCEData? CCEData.harvestDate: null}</td>
        {/* <td className="border-2 pr-2 border-t-0">{cropInformation? cropInformation.cropGrowthStage: null}</td>
        <td className="border-2 pr-2 border-t-0">{cropInformation? cropInformation.croppingPattern: null}</td>
        <td className="border-2 pr-2 border-t-0">{cropInformation? cropInformation.livestock: null}</td>
        <td className="border-2 pr-2 border-t-0">{cropInformation? cropInformation.primaryCrop: null}</td>
        <td className="border-2 pr-2 border-t-0">{cropInformation? cropInformation.primarySeason: null}</td>
        <td className="border-2 pr-2 border-t-0">{cropInformation? cropInformation.secondaryCrop: null}</td>
        <td className="border-2 pr-2 border-t-0">{cropInformation? cropInformation.secondarySeason: null}</td>
        <td className="border-2 pr-2 border-t-0">{cropInformation? cropInformation.remarks: null}</td> */}
</>
    )
}
```

CheckBox.tsx

```
import { useEffect, useState } from "react";
import { useDispatch, useSelector } from "react-redux";
import {
  addDeletionListElement,
  setDeleteAll,
  removeDeletionListElement,
  selectDeleteAll,
} from "../../features/ui";

export enum typeOfCheckBox {
  heading,
  row,
}
export default function ({ type, id }: { type: typeOfCheckBox; id: Number }) {
  const [on, setOn] = useState(false);
  const dispatch = useDispatch();
  const deleteAll = useSelector(selectDeleteAll)
  useEffect(() => {
    if(typeOfCheckBox.row == type){
        if(deleteAll){
            setOn(true);
            dispatch(addDeletionListElement(id));
        }
    }
  }, [deleteAll])
  if (type == typeOfCheckBox.heading) {
    return (
      <td className="flex p-1 items-center gap-2">
        <label htmlFor="" className="p-1 text-lg text-center">Select all</label>
        <input
          type="checkbox"
          checked={deleteAll}
          className="accent-red-500 w-5 h-5"
          onChange={() => {
            if (!deleteAll) {
              dispatch(setDeleteAll(true))
            } else {
              dispatch(setDeleteAll(false));
            }
          }}
          value={deleteAll.toString()}
        />
      </td>
    );
  }
  return (
    <>
      <td className="flex p-1 justify-end px-4">
        <input
          type="checkbox"
          className="accent-red-500 w-5 h-5 ring-1 checked:ring-red-950"
          checked={on}
          onChange={() => {
            setOn((t) => {
              if (!t) {
                dispatch(addDeletionListElement(id));
              } else {
                dispatch(removeDeletionListElement(id));
                dispatch(setDeleteAll(false))
              }
              return !t;
            });
          }}

        />
      </td>
    </>
  );
}

```

basic checkboxes on the table to trigger deletion after selection.

CropInformation.tsx

```
import { cropInfo } from "../SpecificData";

export default function ({ cropInformation }: { cropInformation: cropInfo }) {
  return (
    <>
      <td className="border-2 pr-2 border-t-0">
        {cropInformation ? cropInformation.waterSource : null}
      </td>
      <td className="border-2 pr-2 border-t-0">
        {cropInformation ? cropInformation.cropIntensity : null}
      </td>
      <td className="border-2 pr-2 border-t-0">
        {cropInformation ? cropInformation.cropGrowthStage : null}
      </td>
      <td className="border-2 pr-2 border-t-0">
        {cropInformation ? cropInformation.croppingPattern : null}
      </td>
      {/* <td className="border-2 pr-2 border-t-0">{cropInformation? cropInformation.livestock: null}</td> */}
      <td className="border-2 pr-2 border-t-0">
        {cropInformation ? cropInformation.primaryCrop : null}
      </td>
      <td className="border-2 pr-2 border-t-0">
        {cropInformation ? cropInformation.primarySeason : null}
      </td>
      <td className="border-2 pr-2 border-t-0">
        {cropInformation ? cropInformation.secondaryCrop : null}
      </td>
      <td className="border-2 pr-2 border-t-0">
        {cropInformation ? cropInformation.secondarySeason : null}
      </td>
      <td className="border-2 pr-2 border-t-0">
        {cropInformation ? cropInformation.remarks : null}
      </td>
    </>
  );
}
```

Code to show crop data part in the table.

Footer.tsx

```
import { useDispatch, useSelector } from "react-redux";
import {
  deletionOff,
  emptyList,
  selectDeletion,
  selectDeletionList,
} from "../../features/ui";
import { useCallback } from "react";
import axios from "axios";
import { BACKEND_URL } from "../../App";
import Button from "./Button";


export default function () {
  const deleteOn = useSelector(selectDeletion);
  const dispatch = useDispatch();
  const deletionList = useSelector(selectDeletionList);
  const deleteRequest = useCallback(async () => {
    const response = await axios.post(
      BACKEND_URL + "api/v1/data/deletemany",
      {
        dataId: deletionList,
      },
      {
        headers: {
          Authorization: `Bearer ${localStorage.getItem("token")}`,
        },
      }
    );
    dispatch(deletionOff());
    dispatch(emptyList());
    console.log(response, "is the total response");
  }, [deletionList]);

  const deleteAllRequest = useCallback(async () => {
    await axios.post(
      BACKEND_URL + "api/v1/data/deleteall",
      {
        dataId: deletionList,
      },
      {
        headers: {
          Authorization: `Bearer ${localStorage.getItem("token")}`,
        },
      }
    );
    dispatch(deletionOff());
    dispatch(emptyList());
  }, []);
  if (!deleteOn) null
  if (deleteOn)
    return (
      <div className="justify-center left-1/2 w-fit sticky px-7 py-4 rounded-lg bg-opacity-50 items-center bottom-0 bg-green-500 flex flex-col md:flex-row">
        <div>
          <Button
            className="bg-red-500 p-2 mx-2 rounded-3xl hover:bg-red-700"
            onClick={() => {
              deleteRequest();
            }}
          >
            Delete
          </Button>
          <Button
            className="bg-red-500 p-2 mx-2 rounded-3xl hover:bg-red-700"
            onClick={() => {
              deleteAllRequest();
              dispatch(deletionOff());
              dispatch(emptyList());
            }}
          >
            Delete All
          </Button>
        </div>
      </div>
    );
}

```

Creates button at the footer for deletions.

Navbar.tsx // the important navbar, which has all the buttons and functionalities to delete, help, generate files, archives, sign out, etc.,

```
import { useEffect, useState } from "react";
import { useDispatch, useSelector } from "react-redux";
import { useNavigate } from "react-router-dom";
import {
  deletionOff,
  deletionOn,
  // selectDeleteAll,
  selectDeletion,
  // selectDeletionList,
} from "../../features/ui";
import Button from "./Button";
import ClickableDropdown from "./Modal/ClickableDropdown";
import {
  movePageBack,
  movePageForward,
  selectEntries,
  selectPageNo,
  setEntries,
} from "../../features/data";
import GenerateDataFile from "./Modal/GenerateDataFile";
import DownloadDataWithImages from "./Modal/DownloadDataWithImages";
import { startRoute } from "../../App";
// import axios from "axios";
// import { BACKEND_URL } from "../../App";

export default function () {
  // const [selectableDropdown] = useState(false);
  // const [columns] = useState([]);
  const navigate = useNavigate();
  const deleteOn = useSelector(selectDeletion);
  const dispatch = useDispatch();
  const deletion = useSelector(selectDeletion);
  // const deletionList = useSelector(selectDeletionList);
  const dataAndHandlers = {
    Next: function () {
      return (
        <Button
          onClick={() => {
            dispatch(movePageForward());
          }}
        >
          Next
        </Button>
      );
    },
    Previous: function () {
      return (
        <Button
          onClick={() => {
            dispatch(movePageBack());
          }}
        >
          Previous
        </Button>
      );
    },
    Entries: function () {
      const entries = useSelector(selectEntries);
      const pageNo = useSelector(selectPageNo);
      return (
        <>
          <div className="inline p-2 rounded-sm">
            <label htmlFor="">Entries: </label>
            <input
              type="number"
              className="border-2 max-w-14 p-2 bg-slate-200 rounded-2xl"
              value={entries}
              onChange={(e) => {
                dispatch(setEntries(e.target.value));
              }}
            />
            <label htmlFor="" className="ml-1">
              {" "}
              Page {pageNo}
            </label>
          </div>
        </>
      );
    },
    DownloadDataWithImage: function () {
      const [isModalOpen, setModalOpened] = useState(false);
      return (
        <>
          <Button
            onClick={() => {
              setModalOpened((t) => !t);
            }}
          >
            Download Data with Images
          </Button>
          {isModalOpen ? (
            <DownloadDataWithImages
              closeHandler={() => {
                setModalOpened(false);
              }}
            />
          ) : null}
        </>
      );
    },
    DownloadData: function () {
      const [isModalOpen, setModalOpened] = useState(false);
      return (
        <>
          <Button
            onClick={() => {
              setModalOpened((t) => !t);
            }}
          >
            Download Data File
          </Button>
          {isModalOpen ? (
            <GenerateDataFile
              closeHandler={() => {
                setModalOpened(false);
              }}
            />
          ) : null}
        </>
      );
    },
    Help: function () {
      return (
        <Button
          onClick={() => {
            window.open(
              "https://docs.google.com/document/d/1iokflyiwnFyCEla7k6UOsdiYx3NtD4cgInTQoLP-MBI/edit?usp=sharing",
              "_blank"
            );
          }}
        >
          Help
        </Button>
      );
    },
    Delete: function () {
      return (
        <>
          {!deletion ? (
            <Button
              onClick={() => {
                dispatch(deletionOn());
                // else dispatch(deletionOff())
              }}
            >
              Delete
            </Button>
          ) : (
            <Button
              onClick={() => {
                dispatch(deletionOff());
              }}
            >
              Cancel Deletion
            </Button>
          )}
        </>
      );
    },
    Columns: function () {
      const [isModalOpen, setModalOpened] = useState(false);
      return (
        <>
          <Button
            onClick={() => {
              setModalOpened((t) => !t);
            }}
          >
            Columns
          </Button>
          {isModalOpen ? (
            <ClickableDropdown
              closeHandler={() => setModalOpened(false)}
            ></ClickableDropdown>
          ) : null}
        </>
      );
    },
    Logout: function () {
      return (
        <Button
          onClick={() => {
            localStorage.setItem("token", "");
            localStorage.setItem("email", "");
            navigate(startRoute + "/login");
          }}
        >
          Logout
        </Button>
      );
    },
  };

  useEffect(() => {
    if (
      localStorage.getItem("email") == "" ||
      localStorage.getItem("email") == undefined ||
      localStorage.getItem("token") == "" ||
      localStorage.getItem("token") == undefined
    ) {
      ("/login");
    }
  });
  return (
    <>
      <div className="w-full h-fit bg-green-300 flex justify-between p-2 items-center left-0 sticky top-0">
        <div className="">
          <span>Admin user: {localStorage.getItem("email")}</span>
          {!deleteOn ? (
            <>
              <dataAndHandlers.Next></dataAndHandlers.Next>
              <dataAndHandlers.Previous />
            </>
          ) : null}
          <dataAndHandlers.Entries />
        </div>
        <div className="">
          {!deleteOn ? (
            <>
              <dataAndHandlers.DownloadDataWithImage />
              <dataAndHandlers.DownloadData />
              <dataAndHandlers.Help />
            </>
          ) : null}
          <dataAndHandlers.Delete />
          {!deleteOn ? (
            <>
              <dataAndHandlers.Columns />
              <dataAndHandlers.Logout />
            </>
          ) : null}
        </div>
      </div>
    </>
  );
}
```

TableHeadComponent.tsx

```
// import React from 'react'
import {
  landData,
  cropGrowthStageData,
  cropIntensityData,
  croppingPatternData,
  cropsData,
  // livestockData,
  seasonData,
  waterSourceData,
} from "../dashboard/dataList";
// import { setLandCover } from '../../features/data';

import {
  setAccuracy,
  setBiomassWeight,
  setCropGrowthStage,
  setCropIntensity,
  setCroppingPattern,
  setCultivar,
  setDescription,
  setEmail,
  setHarvestDate,
  setLandCover,
  setLatitude,
  //   setLivestock,
  setLongitude,
  setPrimaryCrop,
  setPrimarySeason,
  setRemarks,
  setSampleSize1,
  setSampleSize2,
  setSecondaryCrop,
  setSecondarySeason,
  setSowDate,
  setWaterSource,
} from "../../features/data";
import CheckBox, { typeOfCheckBox } from "./CheckBox";
import TableHead from "../dashboard/TableHead";
import { useSelector } from "react-redux";
import { selectColumns } from "../../features/ui";
function TableHeadComponent({ deletion }: { deletion: boolean }) {
  const columns = useSelector(selectColumns);
  const tableHeaders = [
    { name: "Latitude", onChangeValueSetter: setLatitude },
    { name: "Longitude", onChangeValueSetter: setLongitude },
    { name: "Accuracy", onChangeValueSetter: setAccuracy },
    {
      name: "Land Cover Type",
      onChangeValueSetter: setLandCover,
      optionsList: landData,
    },
    { name: "Description", onChangeValueSetter: setDescription },
    { name: "By", onChangeValueSetter: setEmail },
    {
      name: "Water Source",
      onChangeValueSetter: setWaterSource,
      optionsList: waterSourceData,
    },
    {
      name: "Crop Intensity",
      onChangeValueSetter: setCropIntensity,
      optionsList: cropIntensityData,
    },
    {
      name: "Crop Growth Stage",
      onChangeValueSetter: setCropGrowthStage,
      optionsList: cropGrowthStageData,
    },
    {
      name: "Cropping Pattern",
      onChangeValueSetter: setCroppingPattern,
      optionsList: croppingPatternData,
    },
    // {
    //   name: "Livestock",
    //   onChangeValueSetter: setLivestock,
    //   optionsList: livestockData,
    // },
    {
      name: "Primary Crop",
      onChangeValueSetter: setPrimaryCrop,
      optionsList: cropsData,
    },
    {
      name: "Primary Season",
      onChangeValueSetter: setPrimarySeason,
      optionsList: seasonData,
    },
    {
      name: "Secondary Crop",
      onChangeValueSetter: setSecondaryCrop,
      optionsList: cropsData,
    },
    {
      name: "Secondary Season",
      onChangeValueSetter: setSecondarySeason,
      optionsList: seasonData,
    },
    { name: "Remarks", onChangeValueSetter: setRemarks },
    { name: "Sample Size 1", onChangeValueSetter: setSampleSize1 },
    { name: "Sample Size 2", onChangeValueSetter: setSampleSize2 },
    { name: "Biomass Weight", onChangeValueSetter: setBiomassWeight },
    { name: "Cultivar", onChangeValueSetter: setCultivar },
    { name: "Sow Date", onChangeValueSetter: setSowDate },
    { name: "Harvest Date", onChangeValueSetter: setHarvestDate },
  ];

  return (
    <tr>
      {deletion ? <CheckBox type={typeOfCheckBox.heading} id={-1} /> : null}
      {tableHeaders.map((header, index) => {
        for (let a of columns) {
          if (a == header.name)
            return (
              <TableHead
                key={index}
                name={header.name}
                onChangeValueSetter={header.onChangeValueSetter}
                optionsList={header.optionsList}
              />
            );
        }
        return null;
      })}
    </tr>
  );
}

export default TableHeadComponent;
```

The top headers row of the table

TableRowComponent.tsx

```
import { useCallback, useEffect, useState } from "react";
import { CCEType, cropInfo } from "../SpecificData";
import CheckBox, { typeOfCheckBox } from "./CheckBox";
import { useNavigate } from "react-router-dom";
import { useSelector } from "react-redux";
import { selectColumns } from "../../features/ui";
import { startRoute } from "../../App";
type Data = {
  id: number;
  latitude: number;
  longitude: number;
  accuracy: number;
  landCover: string;
  description: string;
  cropInformation: cropInfo[];
  CCEdata: CCEType[];
  user: {
    email: string;
  };
  count: Number;
};

function TableRowComponent({
  data,
  deletion,
}: {
  data: null | Data[];
  deletion: boolean;
}) {
  const navigate = useNavigate();
  const columns = useSelector(selectColumns);
  const [columnsVisibility, setColumnsVisibility] = useState([
    {
      name: "Latitude",
      value: false,
    },
    {
      name: "Longitude",
      value: false,
    },
    {
      name: "Accuracy",
      value: false,
    },
    {
      name: "Land Cover Type",
      value: false,
    },
    {
      name: "Description",
      value: false,
    },
    {
      name: "By",
      value: false,
    },
    {
      name: "Water Source",
      value: false,
    },
    {
      name: "Crop Intensity",
      value: false,
    },
    {
      name: "Crop Growth Stage",
      value: false,
    },
    {
      name: "Cropping Pattern",
      value: false,
    },
    {
      name: "Livestock", // DO NOT COMMENT, CODE WILL BREAK
      value: false,
    },
    {
      name: "Primary Crop",
      value: false,
    },
    {
      name: "Primary Season",
      value: false,
    },
    {
      name: "Secondary Crop",
      value: false,
    },
    {
      name: "Secondary Season",
      value: false,
    },
    {
      name: "Remarks",
      value: false,
    },
    {
      name: "Sample Size 1",
      value: false,
    },
    {
      name: "Sample Size 2",
      value: false,
    },
    {
      name: "Biomass Weight",
      value: false,
    },
    {
      name: "Cultivar",
      value: false,
    },
    {
      name: "Sow Date",
      value: false,
    },
    {
      name: "Harvest Date",
      value: false,
    },
  ]);
  const updater = useCallback(() => {
    let tempColumnsVisibility = columnsVisibility;
    tempColumnsVisibility = tempColumnsVisibility.map((v) => {
      return {
        name: v.name,
        value: false,
      };
    });
    for (let a of columns) {
      tempColumnsVisibility = tempColumnsVisibility.map((v) => {
        if (a == v.name || v.value == true) {
          return {
            name: v.name,
            value: true,
          };
        } else
          return {
            name: v.name,
            value: false,
          };
      });
    }
    setColumnsVisibility(tempColumnsVisibility);
  }, [columns]);
  useEffect(() => {
    updater();
    console.log(columnsVisibility);
  }, [columns]);
  if (data != null)
    return (
      <>
        {data.map((value: any) => {
          const newValue = value as Data;
          const CCEData = newValue.CCEdata[0];
          console.log("CCEData", CCEData);
          const CropInfo = newValue.cropInformation[0];
          console.log("Crop Information", CropInfo);
          return (
            <tr>
              {deletion ? (
                <CheckBox type={typeOfCheckBox.row} id={newValue.id} />
              ) : null}
              {columnsVisibility[0].value ? (
                <td className="border-2 pr-2 border-t-0">
                  {newValue.latitude}
                </td>
              ) : null}
              {columnsVisibility[1].value ? (
                <td className="border-2 pr-2 border-t-0">
                  {newValue.longitude}
                </td>
              ) : null}
              {columnsVisibility[2].value ? (
                <td className="border-2 pr-2 border-t-0">
                  {newValue.accuracy}
                </td>
              ) : null}
              {columnsVisibility[3].value ? (
                <td className="border-2 pr-2 border-t-0">
                  {newValue.landCover}
                </td>
              ) : null}
              {columnsVisibility[4].value ? (
                <td className="border-2 pr-2 border-t-0">
                  {newValue.description}
                </td>
              ) : null}
              {columnsVisibility[5].value ? (
                <td className="border-2 pr-2 border-t-0">
                  {newValue.user.email}
                </td>
              ) : null}
              {columnsVisibility[6].value ? (
                <td className="border-2 pr-2 border-t-0">
                  {CropInfo ? CropInfo.waterSource : null}
                </td>
              ) : null}
              {columnsVisibility[7].value ? (
                <td className="border-2 pr-2 border-t-0">
                  {CropInfo ? CropInfo.cropIntensity : null}
                </td>
              ) : null}
              {columnsVisibility[8].value ? (
                <td className="border-2 pr-2 border-t-0">
                  {CropInfo ? CropInfo.cropGrowthStage : null}
                </td>
              ) : null}
              {columnsVisibility[9].value ? (
                <td className="border-2 pr-2 border-t-0">
                  {CropInfo ? CropInfo.croppingPattern : null}
                </td>
              ) : null}
              {/* {columnsVisibility[10].value ? (
                <td className="border-2 pr-2 border-t-0">
                  {CropInfo ? CropInfo.livestock : null}
                </td>
              ) : null} */}
              {columnsVisibility[11].value ? (
                <td className="border-2 pr-2 border-t-0">
                  {CropInfo ? CropInfo.primaryCrop : null}
                </td>
              ) : null}
              {columnsVisibility[12].value ? (
                <td className="border-2 pr-2 border-t-0">
                  {CropInfo ? CropInfo.primarySeason : null}
                </td>
              ) : null}
              {columnsVisibility[13].value ? (
                <td className="border-2 pr-2 border-t-0">
                  {CropInfo ? CropInfo.secondaryCrop : null}
                </td>
              ) : null}
              {columnsVisibility[14].value ? (
                <td className="border-2 pr-2 border-t-0">
                  {CropInfo ? CropInfo.secondarySeason : null}
                </td>
              ) : null}
              {columnsVisibility[15].value ? (
                <td className="border-2 pr-2 border-t-0">
                  {CropInfo ? CropInfo.remarks : null}
                </td>
              ) : null}
              {columnsVisibility[16].value ? (
                <td className="border-2 pr-2 border-t-0">
                  {CCEData ? CCEData.sampleSize_1 : null}
                </td>
              ) : null}
              {columnsVisibility[17].value ? (
                <td className="border-2 pr-2 border-t-0">
                  {CCEData ? CCEData.sampleSize_2 : null}
                </td>
              ) : null}
              {columnsVisibility[18].value ? (
                <td className="border-2 pr-2 border-t-0">
                  {CCEData ? CCEData.biomassWeight : null}
                </td>
              ) : null}
              {columnsVisibility[19].value ? (
                <td className="border-2 pr-2 border-t-0">
                  {CCEData ? CCEData.cultivar : null}
                </td>
              ) : null}
              {columnsVisibility[20].value ? (
                <td className="border-2 pr-2 border-t-0">
                  {CCEData ? CCEData.sowDate : null}
                </td>
              ) : null}
              {columnsVisibility[21].value ? (
                <td className="border-2 pr-2 border-t-0">
                  {CCEData ? CCEData.harvestDate : null}
                </td>
              ) : null}
              {
                <td>
                  <button
                    className="p-3 bg-gray-200 rounded-lg m-2 border-2 border-gray-500 hover:bg-gray-300"
                    onClick={() => {
                      navigate(startRoute + "/data/" + value.id);
                    }}
                  >
                    See more
                  </button>
                </td>
              }
            </tr>
          );
        })}
      </>
    );
}

export default TableRowComponent;
```

The table row component for all other rows.

The `dashboard` folder inside of `pages`

Dashboard.tsx // this brings together the tables, and navbar and the footer.

```
import axios from "axios";
import { useEffect, useState } from "react";
import { useNavigate } from "react-router-dom";
import Spinner from "../Spinner";
import { BACKEND_URL, startRoute } from "../../App";
import Navbar from "../components/Navbar";
import { CCEType, cropInfo } from "../SpecificData";
// import CropInformationData from "../components/CropInformationData";
import { useDispatch, useSelector } from "react-redux";
import { selectData, selectFilterData, setCount } from "../../features/data";

import { selectDeletion, selectDeletionList } from "../../features/ui";
// import CheckBox, { typeOfCheckBox } from "../components/CheckBox";
import Footer from "../components/Footer";
// import CCEInformationData from "../components/CCEInformationData";
import TableHeadComponent from "../components/TableHeadComponent";
import TableRowComponent from "../components/TableRowComponent";

type Data = {
  id: number;
  latitude: number;
  longitude: number;
  accuracy: number;
  landCover: string;
  description: string;
  cropInformation: cropInfo[];
  CCEdata: CCEType[];
  user: {
    email: string;
  };
  count: Number;
};

export default function () {
  const [data, setData] = useState<null | Data[]>(null);
  const overallData = useSelector(selectData);
  const navigate = useNavigate();
  const dispatch = useDispatch();
  const deletion = useSelector(selectDeletion);
  const filterData = useSelector(selectFilterData);
  const deletionList = useSelector(selectDeletionList);
  useEffect(() => {
    if (deletion == true) return;
    if (
      localStorage.getItem("email") == "" ||
      localStorage.getItem("email") == undefined ||
      localStorage.getItem("token") == "" ||
      localStorage.getItem("token") == undefined
    ) {
      navigate(startRoute + "/login");
    }
    async function fetcher() {
      // if(deletionList.length != 0) return
      if (
        filterData.entries == null ||
        filterData.entries == undefined ||
        filterData.entries.toString() == ""
      ) {
        return;
      }
      const response = await axios.post(
        `${BACKEND_URL}api/v1/data/`,
        {
          pageNo: filterData.pageNo,
          entries: filterData.entries,
          latitude: filterData.latitude,
          longitude: filterData.longitude,
          accuracy: filterData.accuracy,
          landCover: filterData.landCover,
          description: filterData.description,
          email: filterData.email,
          sampleSize_1: filterData.sampleSize_1,
          sampleSize_2: filterData.sampleSize_2,
          biomassWeight: filterData.biomassWeight,
          cultivar: filterData.cultivar,
          sowDate: filterData.sowDate,
          harvestDate: filterData.harvestDate,
          waterSource: filterData.waterSource,
          cropIntensity: filterData.cropIntensity,
          primarySeason: filterData.primarySeason,
          primaryCrop: filterData.primaryCrop,
          secondarySeason: filterData.secondarySeason,
          secondaryCrop: filterData.secondaryCrop,
          // livestock: filterData.livestock,
          croppingPattern: filterData.croppingPattern,
          cropGrowthStage: filterData.cropGrowthStage,
          remarks: filterData.remarks,
        },
        {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        }
      );
      if (response.data.logOut) {
        localStorage.setItem("email", "");
        localStorage.setItem("token", "");
        navigate(startRoute + "/login");
      }
      console.log(response.data.count);
      setData(response.data.response);
      console.log(response.data.response);
      dispatch(setCount(response.data.count));
    }
    console.log(deletionList);
    fetcher();
  }, [filterData, deletionList]);
  useEffect(() => {
    console.log("hello");
  }, [overallData]);

  if (data == null) {
    return <Spinner />;
  }

  return (
    <>
      <Navbar></Navbar>
      <div className="text-xl p-4 overflow-y-scroll overflow-x-scroll">
        <TableHeadComponent deletion={deletion}></TableHeadComponent>
        <TableRowComponent data={data} deletion={deletion} />
      </div>
      <Footer></Footer>
    </>
  );
}

```

dataList.tsx
// The data list used at various places.

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

TableHead.tsx

```
import { useDispatch } from "react-redux";

export default function ({
  name,
  onChangeValueSetter,
  optionsList
}: {
  name: String;
  onChangeValueSetter: any;
  optionsList?: {
    title: string;
    value: number
  }[] | null
}) {
  const dispatch = useDispatch();
  return (
    <th className="border-2">
      <span>{name}</span>
      <span className="flex px-2">
        <label htmlFor="" className="font-thin text-lg mx-1">Search:</label>
        <input
          type="text"
          className="border-2 rounded-lg"
          onChange={(e) => {
            dispatch(onChangeValueSetter(e.target.value));
          }}
        />
      </span>
      {
        optionsList? <span className="flex px-2 mt-2">
          <label htmlFor="" className="font-thin text-lg mx-1">Select:</label>
        <select name="cars" id="cars"
        className="bg-red-200 w-full rounded-xl px-2"
        onChange={(e) => {
            console.log(e.target.value)
            if(e.target.value == "Any") {
                dispatch(onChangeValueSetter(null))
                return
            }
            dispatch(onChangeValueSetter(e.target.value))
        }}>
            <option value={"Any"}>Any</option>
            {
                optionsList.map(value => {
                    return (
                        <>
                        <option value={value.title}>{value.title}</option>
                        </>
                    )
                })
            }
        </select>
      </span> :null
      }

    </th>
  );
}
```

Contains the filtering and filter by dropdown selection implementation.

The files at the root level of the `page` folder.

ImageFromFilename.tsx

```
import { BACKEND_URL } from "../App";

export default function ({ filename }: { filename: string }) {
  return (
    <div>
      {
        <img
          src={BACKEND_URL+"api/v1/data/image/" + filename}
          className="w-72 p-2"
          alt="iCrops app image"
        />
      }
    </div>
  );
}
```

a basic image component

Landing.tsx

```
import { useEffect } from "react";
// @ts-ignore
import fileApk from "../app-release.apk";
import { useNavigate } from "react-router-dom";
import { startRoute } from "../App";
export default function () {
  const navigate = useNavigate();
  useEffect(() => {
    if (
      localStorage.getItem("email") != "" &&
      localStorage.getItem("email") != undefined &&
      localStorage.getItem("token") != "" &&
      localStorage.getItem("token") != undefined
    ) {
      navigate(startRoute + "/data");
    }
  }, []);
  return (
    <>
      <div className="w-lvw h-lvh bg-gray-200 flex flex-col items-center justify-center">
        <div className="text-2xl font-bold">Admin Login (iCrops)</div>
        <button
          className="w-4/5 p-3 text-xl rounded-lg bg-white hover:bg-red-100 m-2 md:w-2/5 lg:2/5 xl:1/5"
          onClick={() => {
            navigate(startRoute + "/login");
          }}
        >
          Login
        </button>
        <button
          className="w-4/5 p-3 text-xl rounded-lg bg-white hover:bg-red-100 m-2 md:w-2/5 lg:2/5 xl:1/5"
          onClick={() => {
            navigate(startRoute + "/signup");
          }}
        >
          Sign Up
        </button>
        <a
          href={fileApk}
          className="text-red-950 underline text-lg hover:cursor-pointer hover:text-red-500"
        >
          Download iCrops-app
        </a>
      </div>
    </>
  );
}
```

The basic landing page when the user is not signed in

Login.tsx

```
import { useCallback, useEffect, useState } from "react";
import { useNavigate } from "react-router-dom";
import { BACKEND_URL, startRoute } from "../App";
import axios from "axios";

export default function () {
  const [email, setEmail] = useState<string>("");
  const [password, setPassword] = useState<string>("");
  // const navigate = useNavigate()

  const login = useCallback(async () => {
    console.log(email, password);
    const response = await axios.post(BACKEND_URL + "api/v1/admin/login/", {
      email: email,
      password: password,
    });
    if (response.data.success) {
      localStorage.setItem("token", response.data.jwt);
      localStorage.setItem("email", response.data.email);
      console.log(response.data);
      navigate(startRoute + "/data");
    } else {
      alert("the credentials are wrong");
    }
  }, [email, password]);
  const navigate = useNavigate();
  useEffect(() => {
    if (
      localStorage.getItem("email") != "" &&
      localStorage.getItem("email") != undefined &&
      localStorage.getItem("token") != "" &&
      localStorage.getItem("token") != undefined
    ) {
      navigate(startRoute + "/data");
    }
  }, []);
  return (
    <form
      action=""
      onSubmit={async (e) => {
        e.preventDefault();
        await login();
      }}
      className="w-lvw flex flex-col items-center p-2 justify-center h-lvh bg-gray-100"
    >
      <div className="text-xl mb-5">Admin login to iCrops dashboard</div>
      <label htmlFor="">Email ID</label>
      <input
        required
        type="email"
        value={email}
        className="w-full p-2 my-1 text-xl rounded-lg border-2 md:w-1/2 lg:w-1/3"
        onChange={(e) => {
          setEmail(e.target.value);
        }}
      />
      <label htmlFor="">Password</label>
      <input
        required
        type="password"
        value={password}
        className="w-full p-2 my-1 text-xl rounded-lg border-2 md:w-1/2 lg:w-1/3"
        onChange={(e) => {
          setPassword(e.target.value);
        }}
      />
      <button className="text-xl w-full bg-red-100 p-2 rounded-lg border-2 hover:bg-red-300 md:w-1/2 lg:w-1/3 mt-2">
        Login
      </button>
      <button
        className="w-full md:w-1/2 lg:w-1/3 text-left underline text-red-950 hover:text-red-500"
        onClick={() => {
          navigate(startRoute + "/signup");
        }}
      >
        Sign Up Instead
      </button>
    </form>
  );
}
```

The login page.

SignUp.tsx

```
import axios from "axios";
import { useCallback, useState } from "react";
import { useNavigate } from "react-router-dom";
import { BACKEND_URL, startRoute } from "../App";

export default function () {
  const [email, setEmail] = useState<string>("");
  const [password, setPassword] = useState<string>("");
  const [secret, setSecret] = useState<string>("");
  const [designation, setDesignation] = useState<string>("");
  const [institute, setInstitute] = useState<string>("");
  const [province, setProvince] = useState<string>("");
  const [country, setCountry] = useState<string>("");
  const navigate = useNavigate();
  const login = useCallback(async () => {
    console.log(
      email,
      password,
      password,
      secret,
      designation,
      institute,
      province,
      country
    );

    const response = await axios.post(BACKEND_URL + "api/v1/admin/signup/", {
      email: email,
      password: password,
      secret: secret,
      Designation: designation,
      Country: country,
      Institute: institute,
      Province: province,
    });
    console.log(response);
    if (response?.data.success) navigate(startRoute + "/login");
    else alert("some error");
  }, [email, password, secret, designation, institute, province, country]);
  return (
    <form
      action=""
      onSubmit={(e) => {
        e.preventDefault();
        login();
      }}
      className="w-lvw flex flex-col items-center p-2 justify-center h-lvh bg-gray-100"
    >
      <div className="text-xl mb-5">Sign Up as Admin for iCrops</div>
      <label htmlFor="" className="w-full md:w-1/2 lg:w-1/3 text-left">
        Email ID
      </label>
      <input
        required
        type="email"
        value={email}
        className="w-full p-2 my-1 text-xl rounded-lg border-2 md:w-1/2 lg:w-1/3"
        onChange={(e) => {
          setEmail(e.target.value);
        }}
      />
      <label htmlFor="" className="w-full md:w-1/2 lg:w-1/3 text-left">
        Password
      </label>
      <input
        required
        type="password"
        value={password}
        className="w-full p-2 my-1 text-xl rounded-lg border-2 md:w-1/2 lg:w-1/3"
        onChange={(e) => {
          setPassword(e.target.value);
        }}
      />
      <label htmlFor="" className="w-full md:w-1/2 lg:w-1/3 text-left">
        Designation
      </label>
      <input
        required
        type="text"
        value={designation}
        className="w-full p-2 my-1 text-xl rounded-lg border-2 md:w-1/2 lg:w-1/3"
        onChange={(e) => {
          setDesignation(e.target.value);
        }}
      />
      <label htmlFor="" className="w-full md:w-1/2 lg:w-1/3 text-left">
        Institute
      </label>
      <input
        required
        type="text"
        value={institute}
        className="w-full p-2 my-1 text-xl rounded-lg border-2 md:w-1/2 lg:w-1/3"
        onChange={(e) => {
          setInstitute(e.target.value);
        }}
      />
      <label htmlFor="" className="w-full md:w-1/2 lg:w-1/3 text-left">
        Province
      </label>
      <input
        required
        type="text"
        value={province}
        className="w-full p-2 my-1 text-xl rounded-lg border-2 md:w-1/2 lg:w-1/3"
        onChange={(e) => {
          setProvince(e.target.value);
        }}
      />
      <label htmlFor="" className="w-full md:w-1/2 lg:w-1/3 text-left">
        Country
      </label>
      <input
        required
        type="text"
        value={country}
        className="w-full p-2 my-1 text-xl rounded-lg border-2 md:w-1/2 lg:w-1/3"
        onChange={(e) => {
          setCountry(e.target.value);
        }}
      />
      <label htmlFor="" className="w-full md:w-1/2 lg:w-1/3 text-left">
        Authentication Secret
      </label>
      <input
        required
        type="password"
        value={secret}
        className="w-full p-2 my-1 text-xl rounded-lg border-2 md:w-1/2 lg:w-1/3"
        onChange={(e) => {
          setSecret(e.target.value);
        }}
      />
      <button className="text-xl w-full bg-red-100 p-2 rounded-lg border-2 hover:bg-red-300 md:w-1/2 lg:w-1/3 mt-2">
        Sign Up
      </button>
      <button
        className="w-full md:w-1/2 lg:w-1/3 text-left underline text-red-950 hover:text-red-500"
        onClick={() => {
          navigate(startRoute + "/login");
        }}
      >
        Login Instead
      </button>
    </form>
  );
}
```

The sign up page.

SpecificData.tsx

```
import axios from "axios";
import { useEffect, useState } from "react";
import { useNavigate, useParams } from "react-router-dom";
import ImageFromFilename from "./ImageFromFilename";
import Spinner from "./Spinner";
import { BACKEND_URL, startRoute } from "../App";
import Navbar from "./components/Navbar";

interface imgType {
  id: number;
  fileName: string;
  dataId: number;
}

export interface cropInfo {
  waterSource: string;
  cropGrowthStage: string;
  cropIntensity: string;
  // livestock: string;
  croppingPattern: string;
  primaryCrop: string;
  primarySeason: string;
  remarks: string;
  secondaryCrop: string;
  secondarySeason: string;
}

export interface CCEType {
  biomassWeight: string;
  cultivar: string;
  grainWeight: string;
  dataId: number;
  harvestDate: string;
  id: number;
  sampleSize_1: number;
  sampleSize_2: number;
  sowDate: string;
}

export default function () {
  const { dataid } = useParams();
  const navigate = useNavigate();
  const [data, setData] = useState<{
    latitude: number;
    longitude: number;
    accuracy: number;
    landCover: string;
    cropInformation: cropInfo[];
    CCEdata: CCEType[];
    images: imgType[];
    user: {
      email: string;
    };
  } | null>(null);
  useEffect(() => {
    if (
      localStorage.getItem("email") == "" ||
      localStorage.getItem("email") == undefined ||
      localStorage.getItem("token") == "" ||
      localStorage.getItem("token") == undefined
    ) {
      navigate(startRoute + "/login");
    }
    async function fetchAnEntry(dataId: number) {
      console.log();
      const response = await axios.post(
        BACKEND_URL + "api/v1/data/" + dataId + "/",
        {},
        {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        }
      );
      return response.data;
    }
    console.log(dataid);

    if (dataid)
      fetchAnEntry(parseInt(dataid)).then((val) => {
        setData(val);
        console.log(val);
      });
  }, []);
  if (data == null) return <Spinner />;
  return (
    <div>
      <Navbar></Navbar>
      {data ? (
        <div className="m-2">
          <button
            className="border-gray-500 p-3 rounded-lg bg-gray-200 hover:bg-gray-300 border-2 mb-2"
            onClick={() => {
              navigate(startRoute + "/data");
            }}
          >
            Go Back
          </button>
          <div>
            <span className="font-bold">By: </span> {data.user.email}
          </div>
          <div>
            <span className="font-bold">Latitude</span>: {data.latitude}
          </div>
          <div>
            <span className="font-bold">Longitude</span>: {data.longitude}
          </div>
          <div>
            <span className="font-bold">Accuracy</span>: {data.accuracy}
          </div>
          <div>
            <span className="font-bold">Land Cover</span>: {data.landCover}
          </div>
          {data.cropInformation?.length > 0 ? (
            <div>
              <h3 className="font-bold mt-4 mb-2">Crop Information</h3>
              <div>
                <span className="font-bold">Water Source</span>:{" "}
                {data.cropInformation[0].waterSource}
              </div>
              <div>
                <span className="font-bold">Crop growth stage</span>:{" "}
                {data.cropInformation[0].cropGrowthStage}
              </div>
              <div>
                <span className="font-bold">Crop intensity</span>:{" "}
                {data.cropInformation[0].cropIntensity}
              </div>
              {/* <div>
                <span className="font-bold">Livestock</span>:{" "}
                {data.cropInformation[0].livestock}
              </div> */}
              <div>
                <span className="font-bold">Cropping Pattern</span>:{" "}
                {data.cropInformation[0].croppingPattern}
              </div>
              <div className="font-bold mt-3">Season 1</div>
              <div>
                <span className="font-bold">Crop</span>:{" "}
                {data.cropInformation[0].primaryCrop}
              </div>
              <div>
                <span className="font-bold">Season</span>:{" "}
                {data.cropInformation[0].primarySeason}
              </div>
              <div className="font-bold mt-3">Season 2</div>
              <div>
                <span className="font-bold">Crop</span>:{" "}
                {data.cropInformation[0].secondaryCrop}
              </div>
              <div>
                <span className="font-bold">Season</span>:{" "}
                {data.cropInformation[0].secondarySeason}
              </div>
              <div>
                <span className="font-bold mt-2">Remarks</span>:{" "}
                {data.cropInformation[0].remarks}
              </div>
            </div>
          ) : null}
          {data.CCEdata?.length > 0 ? (
            <div>
              <h3 className="mt-3 font-bold">CCE Data</h3>
              <div>
                <span className="font-bold">Biomass Weight</span>:{" "}
                {data.CCEdata[0].biomassWeight} kg
              </div>
              <div>
                <span className="font-bold">Sample Size</span>:{" "}
                {data.CCEdata[0].sampleSize_1} m X{" "}
                {data.CCEdata[0].sampleSize_2} m
              </div>
              <div>
                <span className="font-bold">Grain Weight</span>:{" "}
                {data.CCEdata[0].grainWeight} kg
              </div>
              <div>
                <span className="font-bold">Harvest Date</span>:{" "}
                {data.CCEdata[0].harvestDate.toString()}
              </div>
              <div>
                <span className="font-bold">Sow Date</span>:{" "}
                {data.CCEdata[0].sowDate.toString()}
              </div>
            </div>
          ) : null}
          <h3 className="font-bold mt-4">Images</h3>
          <div className="flex flex-col md:flex-row w-lvw p-2 pt-0">
            {data.images?.length > 0 &&
              (data.images.map((image) => (
                <ImageFromFilename key={image.id} filename={image.fileName} />
              )) ||
                "nothing")}
          </div>
        </div>
      ) : null}
    </div>
  );
}
```

The page to show each data in detail with images.

Spinner.tsx

```
export default function () {
    return (
<div className="w-lvw h-lvh flex justify-center items-center bg-gray-300">

<div role="status">
    <svg aria-hidden="true" className="inline w-10 h-10 text-gray-200 animate-spin dark:text-gray-600 fill-blue-600" viewBox="0 0 100 101" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z" fill="currentColor"/>
        <path d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z" fill="currentFill"/>
    </svg>
    <span className="sr-only">Loading...</span>
</div>
</div>

    )
}
```

The loading animation used.
