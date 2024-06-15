# The iCrops-backend server

This is the backend server for the iCrops mobile application. It is built using Node.js, and uses Typescript as its programming language.

It also uses Python to generate archives, and to generate the `.shp` files.

You can understand the other dependencies used by this app using the the `package.json` below

```
{
  "name": "icrops-backend",
  "version": "1.0.0",
  "description": "",
  "main": "dist/index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "npx prisma generate && npx prisma migrate deploy && tsc --b && cp ./src/v1/routes/file/main.py ./dist/v1/routes/file/main.py && node dist/index.js",
    "build-and-run": "npx prisma generate && npx prisma migrate deploy && npx tsc --build && copy .\\src\\v1\\routes\\file\\main.py .\\dist\\v1\\routes\\file\\main.py && copy .\\src\\v1\\routes\\archiveGenerator\\archiver.py .\\dist\\v1\\routes\\archiveGenerator\\archiver.py && node dist\\index.js",
    "start": "node dist\\index.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "@prisma/client": "^5.15.0",
    "@types/adm-zip": "^0.5.5",
    "@types/cors": "^2.8.17",
    "@types/express": "^4.17.21",
    "@types/jsonwebtoken": "^9.0.6",
    "adm-zip": "^0.5.12",
    "archiver": "^7.0.1",
    "body-parser": "^1.20.2",
    "cors": "^2.8.5",
    "csv-stringify": "^6.4.6",
    "csv-writer": "^1.6.0",
    "exiftool-vendored": "^26.0.0",
    "express": "^4.19.2",
    "jsonwebtoken": "^9.0.2",
    "rimraf": "^5.0.7",
    "ts-node": "^10.9.2",
    "typescript": "^5.4.3",
    "write-excel-file": "^1.4.30",
    "zod": "^3.22.4"
  },
  "devDependencies": {
    "@types/node": "^20.12.12",
    "prisma": "^5.15.0"
  }
}
```

You can also see, that to run this project, we first do `npm install` then we do `npm run dev` at linux distros or `npm run build-and-run` at windows.

Or you can simply start the `server-start.bat` script which keeps the server alive even if the server crashes, the `bash` script can also be written in a similar way.

server-start.bat

```
@echo off
setlocal

REM Set the path to the Node.js script
set NODE_SCRIPT=dist/index.js

REM Set the interval for checking (in seconds)
set CHECK_INTERVAL=5

REM Set the initial command to start the server
set START_COMMAND=npm run build-and-run

:LOOP
REM Check if the Node.js process is running
tasklist /FI "IMAGENAME eq node.exe" /FI "WINDOWTITLE eq %NODE_SCRIPT%" 2>NUL | find /I /N "node.exe">NUL
if "%ERRORLEVEL%"=="0" (
    echo Node.js process is running.
) else (
    echo Node.js process is not running. Restarting...
    REM Start the Node.js server using the appropriate command
    call %START_COMMAND%
    echo Server restarted.
    REM Switch to npm run start for subsequent restarts
    set START_COMMAND=npm run start
)

REM Wait for the specified interval before checking again
timeout /t %CHECK_INTERVAL% /nobreak >NUL

REM Continue the loop
goto LOOP
```

default_nginx has the nginx configuration file, if using nginx as the reverse proxy server at linux.

# The code and structure

```
- prisma
- savedImages // all the images uploaded to the server are stored here.
- src // contains all the main code
    - v1
        - fileIntercept
            index.ts
        - routes
            - admin
                index.ts
            - archiveGeneratoe
                archiver.py
                index.ts
                xlsx.ts
            - data
                fetcher.ts
                filter.ts
                index.ts
            - file
                index.ts
                main.py
                shp.ts
                xlsx.ts
            - sync
                imageMetaDataSetter.ts
                index.ts
                single.ts
            - user
                index.ts
            - transaction
                transaction.ts
    index.ts
```

As this is `TypeScript` the moment you run the server using the above commands, the typescript is compiled to javascript in the `dist` folder, and then the server runs from those files, meaning `node_modules` and `dist` folder can be completely deleted and the rest can be shared without any issues.

- `index.ts`
  This is the entry point of all the `routes` routes of the backend.

```
import express from "express";

import initialVersionRouter from "./v1/routes";
import { PrismaClient } from "@prisma/client";
import cors from "cors";
import https from "node:https"
import fs from "node:fs"
import path from "node:path"
import { readFileSync } from "fs";
import bodyParser from "body-parser";
const prisma = new PrismaClient();
const app = express()

// app.use((req, res, next) => {
//   res.header('Access-Control-Allow-Origin', 'https://dit2dtt5nci8z.cloudfront.net/  ');
//   res.header('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
//   res.header('Access-Control-Allow-Headers', 'Origin, X-Requested-With, Content-Type, Accept');
//   next();
// });

app.use(bodyParser.json({ limit: '20mb' }));
app.use(bodyParser.urlencoded({ limit: '20mb', extended: true }));

app.use(cors())



app.use("/api/v1", initialVersionRouter);

app.get("/", (req, res) => {
  console.log("it works")
  res.send("hello")
})

const server = app.listen(8090, () => {
  console.log("listening on port 8090");
  server.setTimeout(60000)
})
```

We now go into the `src/v1` folder and start exploring the code.

- `fileIntercept` folder
  index.ts

```
import path from "node:path"
import fs from "fs"
import { setGPSMetadata } from "../routes/sync/imageMetaDataSetter";
import { PrismaClient } from "@prisma/client";
const prisma = new PrismaClient();
export  function saveBase64File(base64String:string, fileName:string, latitude: number|undefined, longitude: number|undefined) {
    // Remove the header from the base64 string
    const base64Data = base64String.replace(/^data:\w+\/\w+;base64,/, '');


    // Convert base64 string to binary buffer
    const buffer = Buffer.from(base64Data, 'base64');

    // Construct the file path
    const filePath = path.join(__dirname, "..", "..", "..", "savedImages", fileName)
    // const filePath = __dirname+'/'+folderPath + '/' + fileName;

    // Write buffer to file
    fs.writeFile(filePath, buffer, async (err) => {
        if (err) {
            console.error('Error saving file:', err);
        } else {
            console.log('File saved successfully:', filePath);
            // const data = await prisma.data.findFirst()
            await setGPSMetadata(filePath, latitude, longitude);
        }
    });
}

```

This code is used to save the image that was uploaded, as the image is uploaded using the `base64` encoding, so it must be decoded and saved as the jpeg file in the `savedImages` folder at the root.

- the `transactions` folder.

```
import { PrismaClient } from "@prisma/client";
import fs from "node:fs";
import path from "node:path";

const prisma = new PrismaClient();

interface TransactionTypeFace {
    userId: number; // it is from random
    data: DataType;
    images: {
        imgList: string[],
        syncCount: number,
        totalImages: number,
    };
}

interface DataType {
    capturedFromMaps: boolean;
    latitude: number;
    longitude: number;
    accuracyCorrection: number;
    bearingToCenter: number;
    distanceToCenter: number[],
    landCoverType: string,
    cropInformation: {
        isCaptured: boolean,
        waterSource: string,
        cropIntensity: string,
        primarySeason: { seasonName: string, cropName: string },
        secondarySeason: { seasonName: string, cropName: string },
        // liveStock: string,
        croppingPattern: string,
        cropGrowthStage: string,
        remarks: string
    },
    CCE: {
        isCaptured: boolean,
        isGoingToBeCaptured: boolean,
        sampleSize: number,
        grainWeight: string
        biomassWeight: string,
        cultivar: string,
        sowDate: Date,
        harvestDate: Date,
        sampleSize1: string,
        sampleSize2: string
    },
    images: string[]
    locationDesc: null | string,
    noOfImages: number
}

export class Transactions {
    private static instance: Transactions;
    private dataRecords: Record<string, TransactionTypeFace>;
    private constructor() {
        this.dataRecords = {}
    }
    static getInstance() {
        console.log("hello")
        if (!Transactions.instance) {
            Transactions.instance = new Transactions();
            return Transactions.instance;
        }
        else return Transactions.instance;
    }

    ifDataExists(dataId: string): boolean {
        if (this.dataRecords[dataId].data) {
            return true;
        }
        return false;
    }

    latitudeAndLongitude(dataId: string): {
        latitude: number;
        longitude: number;
    } {
        return {
            latitude: this.dataRecords[dataId].data.latitude,
            longitude: this.dataRecords[dataId].data.longitude
        }
    }

    insertData(data: DataType, userId: number): { dataId: string } {
        const dataId = crypto.randomUUID();
        this.dataRecords[dataId] = {
            data: data,
            images: {
                imgList: [],
                syncCount: 0,
                totalImages: data.images.length
            },
            userId: userId
        };
        this.dataRecords[dataId]
        console.log("data id is", dataId)
        return { dataId };
    }

    syncImageUpdate(dataId: number, imgName: string) {
        this.dataRecords[dataId].images.syncCount++;
        this.dataRecords[dataId].images.imgList.push(imgName)
    }

    async checkAndPush(dataId: number) {
        if (this.dataRecords[dataId].images.syncCount != this.dataRecords[dataId].images.totalImages) return;
        // this.dataRecords[dataId] //-> push this to db now
        const body = this.dataRecords[dataId].data
        if (body) {
            if (body.latitude != null && body.longitude != null && body.accuracyCorrection != null && body.landCoverType != null) {
                if (body.landCoverType == "Cropland") {
                    if (body.cropInformation.cropGrowthStage != null && body.cropInformation.cropIntensity != null && body.cropInformation.croppingPattern != null && body.cropInformation.isCaptured != null &&
                        // body.cropInformation.liveStock != null &&
                        body.cropInformation.primarySeason.cropName != null && body.cropInformation.primarySeason.seasonName != null && body.cropInformation.secondarySeason.cropName != null && body.cropInformation.secondarySeason.seasonName != null) {
                        console.log("hello", "NO ERRPR")
                    }
                    else return;
                    if (body.CCE.isCaptured) {
                        if (body.CCE.biomassWeight != null && body.CCE.cultivar != null && body.CCE.grainWeight != null && body.CCE.harvestDate != null && body.CCE.sampleSize1 != null && body.CCE.sampleSize2 != null && body.CCE.sowDate != null) { }
                        else return;
                    }
                }
                if (body.images.length > 0) { }
                else return;
            }
            else return;
        }
        else return;

        const result = await prisma.data.create({
            data: {
                latitude: body.latitude,
                longitude: body.longitude,
                accuracy: body.accuracyCorrection,
                landCover: body.landCoverType,
                description: body.locationDesc || " ",
                //   userId: 2,
                imageCount: body.noOfImages,
                user: {
                    connect: { id: this.dataRecords[dataId].userId },
                },
            },
        });
        if (body.landCoverType == "Cropland") {
            const cropInformation = body.cropInformation;
            await prisma.cropInformation.create({
                data: {
                    cropGrowthStage: cropInformation.cropGrowthStage || " ",
                    cropIntensity: cropInformation.cropIntensity || " ",
                    croppingPattern: cropInformation.croppingPattern || " ",
                    livestock: "cropInformation.liveStock" || " ",
                    primaryCrop: cropInformation.primarySeason.cropName || " ",
                    primarySeason: cropInformation.primarySeason.seasonName || " ",
                    remarks: cropInformation.remarks || " ",
                    secondaryCrop: cropInformation.secondarySeason.cropName || " ",
                    secondarySeason: cropInformation.secondarySeason.seasonName || " ",
                    waterSource: cropInformation.waterSource || " ",
                    data: {
                        connect: {
                            id: result.id,
                        },
                    },
                },
            });
        }
        if (body.CCE.isCaptured) {
            const CCEData = body.CCE;
            await prisma.cCE.create({
                data: {
                    biomassWeight: parseInt(CCEData.biomassWeight) || 0,
                    cultivar: CCEData.cultivar || "",
                    sowDate: new Date(CCEData.sowDate) || "",
                    grainWeight: parseInt(CCEData.grainWeight) || 0,
                    harvestDate: new Date(CCEData.harvestDate) || "",
                    sampleSize_1: parseInt(CCEData.sampleSize1) || 0,
                    sampleSize_2: parseInt(CCEData.sampleSize2) || 0,
                    data: {
                        connect: {
                            id: result.id,
                        },
                    },
                },
            });
        }
        for (let fileName of this.dataRecords[dataId].images.imgList) {
            const imageResult = await prisma.images.create({
                data: {
                    fileName: fileName,
                    data: {
                        connect: {
                            id: result.id,
                        },
                    },
                },
            });
        }
        console.log(result);
        const response = await prisma.integrity.create({
            data: {
                timeAdded: new Date(),
                timerId: 0,
                dataId: result.id,
                complete: false
            }
        })


        //destruct itself now
        delete this.dataRecords[dataId];
    }

    destructAndClean(dataId: string) {
        setTimeout(async () => {
            if (!this.dataRecords[dataId]) return;
            if (this.dataRecords[dataId].images.syncCount == this.dataRecords[dataId].images.totalImages) return;
            else {
                console.log("deletion fired");

                for (let a of this.dataRecords[dataId].images.imgList) {
                    // delete the images of a name
                    fs.unlinkSync(path.join(__dirname, "..", "..", "..", "savedImages", a));
                    fs.unlinkSync(path.join(__dirname, "..", "..", "..", "savedImages", a + "_original"));
                    console.log(a, "deleted");
                }
                delete this.dataRecords[dataId];
            }
        }, 720000);
    }

};
```

This code is used to sync the data from the native-apps, and is atomic, meaning at the moment the last image route is hit and image is uploaded, and if all the data that was sent before is correct, then the entry is made in the database, and the data is persisted. Earlier while the transaction is still going on, we store it in memory with a timeout of approximately 12 minutes, after which that incomplete data will be flushed from the memory of the server. It uses the singleton design to accomplish this using classes and objects.

This means that _each entry_ must take 12 minutes or less to upload in total.
Either ways it is not good to increase it more, otherwise the slow bandwidth users will choke the server and not allow for others to sync their data, as a single machine server has its own bandwidth limitations.

This way, either a complete entry is made to the database, or no entry is made and the images are deleted meaning it rolled back, and an error response is set, and the data still stays in the native-app and can be tried to resync later.

```
import { PrismaClient } from "@prisma/client";
import fs from "node:fs";
import path from "node:path";

const prisma = new PrismaClient();

interface TransactionTypeFace {
    userId: number; // it is from random
    data: DataType;
    images: {
        imgList: string[],
        syncCount: number,
        totalImages: number,
    };
}

interface DataType {
    capturedFromMaps: boolean;
    latitude: number;
    longitude: number;
    accuracyCorrection: number;
    bearingToCenter: number;
    distanceToCenter: number[],
    landCoverType: string,
    cropInformation: {
        isCaptured: boolean,
        waterSource: string,
        cropIntensity: string,
        primarySeason: { seasonName: string, cropName: string },
        secondarySeason: { seasonName: string, cropName: string },
        // liveStock: string,
        croppingPattern: string,
        cropGrowthStage: string,
        remarks: string
    },
    CCE: {
        isCaptured: boolean,
        isGoingToBeCaptured: boolean,
        sampleSize: number,
        grainWeight: string
        biomassWeight: string,
        cultivar: string,
        sowDate: Date,
        harvestDate: Date,
        sampleSize1: string,
        sampleSize2: string
    },
    images: string[]
    locationDesc: null | string,
    noOfImages: number
}

export class Transactions {
    private static instance: Transactions;
    private dataRecords: Record<string, TransactionTypeFace>;
    private constructor() {
        this.dataRecords = {}
    }
    static getInstance() {
        console.log("hello")
        if (!Transactions.instance) {
            Transactions.instance = new Transactions();
            return Transactions.instance;
        }
        else return Transactions.instance;
    }

    ifDataExists(dataId: string): boolean {
        if (this.dataRecords[dataId].data) {
            return true;
        }
        return false;
    }

    latitudeAndLongitude(dataId: string): {
        latitude: number;
        longitude: number;
    } {
        return {
            latitude: this.dataRecords[dataId].data.latitude,
            longitude: this.dataRecords[dataId].data.longitude
        }
    }

    insertData(data: DataType, userId: number): { dataId: string } {
        const dataId = crypto.randomUUID();
        this.dataRecords[dataId] = {
            data: data,
            images: {
                imgList: [],
                syncCount: 0,
                totalImages: data.images.length
            },
            userId: userId
        };
        this.dataRecords[dataId]
        console.log("data id is", dataId)
        return { dataId };
    }

    syncImageUpdate(dataId: number, imgName: string) {
        this.dataRecords[dataId].images.syncCount++;
        this.dataRecords[dataId].images.imgList.push(imgName)
    }

    async checkAndPush(dataId: number) {
        if (this.dataRecords[dataId].images.syncCount != this.dataRecords[dataId].images.totalImages) return;
        // this.dataRecords[dataId] //-> push this to db now
        const body = this.dataRecords[dataId].data
        if (body) {
            if (body.latitude != null && body.longitude != null && body.accuracyCorrection != null && body.landCoverType != null) {
                if (body.landCoverType == "Cropland") {
                    if (body.cropInformation.cropGrowthStage != null && body.cropInformation.cropIntensity != null && body.cropInformation.croppingPattern != null && body.cropInformation.isCaptured != null &&
                        // body.cropInformation.liveStock != null &&
                        body.cropInformation.primarySeason.cropName != null && body.cropInformation.primarySeason.seasonName != null && body.cropInformation.secondarySeason.cropName != null && body.cropInformation.secondarySeason.seasonName != null) {
                        console.log("hello", "NO ERRPR")
                    }
                    else return;
                    if (body.CCE.isCaptured) {
                        if (body.CCE.biomassWeight != null && body.CCE.cultivar != null && body.CCE.grainWeight != null && body.CCE.harvestDate != null && body.CCE.sampleSize1 != null && body.CCE.sampleSize2 != null && body.CCE.sowDate != null) { }
                        else return;
                    }
                }
                if (body.images.length > 0) { }
                else return;
            }
            else return;
        }
        else return;

        const result = await prisma.data.create({
            data: {
                latitude: body.latitude,
                longitude: body.longitude,
                accuracy: body.accuracyCorrection,
                landCover: body.landCoverType,
                description: body.locationDesc || " ",
                //   userId: 2,
                imageCount: body.noOfImages,
                user: {
                    connect: { id: this.dataRecords[dataId].userId },
                },
            },
        });
        if (body.landCoverType == "Cropland") {
            const cropInformation = body.cropInformation;
            await prisma.cropInformation.create({
                data: {
                    cropGrowthStage: cropInformation.cropGrowthStage || " ",
                    cropIntensity: cropInformation.cropIntensity || " ",
                    croppingPattern: cropInformation.croppingPattern || " ",
                    livestock: "cropInformation.liveStock" || " ",
                    primaryCrop: cropInformation.primarySeason.cropName || " ",
                    primarySeason: cropInformation.primarySeason.seasonName || " ",
                    remarks: cropInformation.remarks || " ",
                    secondaryCrop: cropInformation.secondarySeason.cropName || " ",
                    secondarySeason: cropInformation.secondarySeason.seasonName || " ",
                    waterSource: cropInformation.waterSource || " ",
                    data: {
                        connect: {
                            id: result.id,
                        },
                    },
                },
            });
        }
        if (body.CCE.isCaptured) {
            const CCEData = body.CCE;
            await prisma.cCE.create({
                data: {
                    biomassWeight: parseInt(CCEData.biomassWeight) || 0,
                    cultivar: CCEData.cultivar || "",
                    sowDate: new Date(CCEData.sowDate) || "",
                    grainWeight: parseInt(CCEData.grainWeight) || 0,
                    harvestDate: new Date(CCEData.harvestDate) || "",
                    sampleSize_1: parseInt(CCEData.sampleSize1) || 0,
                    sampleSize_2: parseInt(CCEData.sampleSize2) || 0,
                    data: {
                        connect: {
                            id: result.id,
                        },
                    },
                },
            });
        }
        for (let fileName of this.dataRecords[dataId].images.imgList) {
            const imageResult = await prisma.images.create({
                data: {
                    fileName: fileName,
                    data: {
                        connect: {
                            id: result.id,
                        },
                    },
                },
            });
        }
        console.log(result);
        const response = await prisma.integrity.create({
            data: {
                timeAdded: new Date(),
                timerId: 0,
                dataId: result.id,
                complete: false
            }
        })


        //destruct itself now
        delete this.dataRecords[dataId];
    }

    destructAndClean(dataId: string) {
        setTimeout(async () => {
            if (!this.dataRecords[dataId]) return;
            if (this.dataRecords[dataId].images.syncCount == this.dataRecords[dataId].images.totalImages) return;
            else {
                console.log("deletion fired");

                for (let a of this.dataRecords[dataId].images.imgList) {
                    // delete the images of a name
                    fs.unlinkSync(path.join(__dirname, "..", "..", "..", "savedImages", a));
                    fs.unlinkSync(path.join(__dirname, "..", "..", "..", "savedImages", a + "_original"));
                    console.log(a, "deleted");
                }
                delete this.dataRecords[dataId];
            }
        }, 720000);
    }

};

```

- The `routes` folder

index.ts, it is the entry point for all the `routes` and the routing system can be understood well from here.

```
import { Router } from "express";
import { singleSyncRouter } from "./sync";
import dataRouter from "./data";
import userRouter from "./user";
import adminRouter from "./admin";
import fileDataRouter from "./file";
import archiveRouter from "./archiveGenerator";

const initialVersionRouter = Router();

initialVersionRouter.get("/", (req, res) => {
    console.log("v1 router")
    res.send("from v1")
})

initialVersionRouter.use("/sync", singleSyncRouter);
initialVersionRouter.use('/file-data', fileDataRouter);
initialVersionRouter.use('/archive-data', archiveRouter);
initialVersionRouter.use("/data", dataRouter);
initialVersionRouter.use("/admin", adminRouter)
initialVersionRouter.use("/user", userRouter);

export default initialVersionRouter
```

`admin`/index.ts , this has all the routes of authentication and middlewares for admin, ie., the accounts used to view data at the frontend-react dashboard. It takes a special field entry for creation of this type of user, this can be used to sign in to the native-app as well as the frontend-react app. The auth-key is required, and currently it is set to `123`

```
import { PrismaClient } from "@prisma/client";
import { NextFunction, Request, Response, Router } from "express";
import pkg from "jsonwebtoken";
import { JWT_SECRET } from "../user";
const { sign, verify } = pkg;
const adminRouter = Router()
// @ts-ignore
const ADMIN_SECRET:string = process.env.ADMIN_SECRET;
const prisma = new PrismaClient()

declare global {
    namespace Express {
      interface Request {
        adminId?: number;
      }
    }
  }

export const adminAuthMiddleware = async (
    req: Request,
    res: Response,
    next: NextFunction
  ) => {
    const authHeader: string | undefined = req.headers.authorization;
    const jwt = authHeader?.split(" ")[1];
    if (jwt)
      try {
        const data = await verify(jwt, JWT_SECRET);
        // @ts-ignore
        const id = data.id;
        console.log(data, "user id");
        // next();
        const response = await prisma.user.findUniqueOrThrow({
          where: {
            id: id,
            admin: true
          },
        });
        console.log(response, "admin found with that jwt");
        req.adminId = id;
        next();
      } catch (e) {
        res.json({
          success: false,
          logOut: true
        });
        console.log(e);
      }
  };

adminRouter.post("/login", async (req, res) => {
    const body: {
        email: string;
        password: string;
      } = req.body;
      try {
        const response = await prisma.user.findUniqueOrThrow({
          where: {
            email: body.email,
            password: body.password,
            admin: true
          },
        });
        console.log(response, " FROM THE DB");
        const jwt = await sign({ id: response.id }, JWT_SECRET);
        res.send({ jwt, success: true, email: response.email });
      } catch (e) {
        console.log(e);
        res.send({ success: false });
      }
})

adminRouter.post("/signup", async (req, res) => {
    const body: {
      email: string;
      password: string;
      Designation: string;
      Institute: string;
      Province: string;
      Country: string;
      username: string;
      secret: string;
    } = req.body;

    try {
      if(body.secret != ADMIN_SECRET) throw Error();
      const response = await prisma.user.create({
        data: {
          Country: body.Country,
          Designation: body.Designation,
          email: body.email,
          Institute: body.Institute,
          password: body.password,
          Province: body.Province,
          admin: true,
          synced: 0
        },
      });
      console.log(response);
      res.json({
        success: true,
      });
    } catch (e) {
      console.log(e);
      res.json({
        success: false,
      });
    }
  });


export default adminRouter;
```

`archiveGenerator` folder

archiver.py, to generate archives.

```
import os
import sys
import shutil

def zip_folder(folder_path, output_path):
    """
    Zip the contents of an entire folder (with that folder included
    in the archive). Empty sub-folders will be included in the archive
    as well.

    :param folder_path: The folder to zip.
    :param output_path: The path to the output zip file.
    """
    shutil.make_archive(output_path,'zip', folder_path)


if __name__ == "__main__":
    # Ensure both CSV filename and target folder name are provided
    if len(sys.argv) != 3:
        print("Usage: python archiver.py <folder_path> <output_file>")
        sys.exit(1)

    # Extract command line arguments
    folder_path = sys.argv[1]
    output_file = sys.argv[2]

    # Create the target folder if it doesn't exist
    # if not os.path.exists(target_folder):
    #     os.makedirs(target_folder)

    # Convert CSV to Shapefile
    zip_folder(folder_path, output_file)

```

Normally the command lines of linux has a built in tool, and is significantly faster, but to support cross platform, where in issues are created at windows like OS we use a python child process to make the archive for us.

index.ts, this is the basic routing of the archive generation

```
import { Request, Response, Router } from "express";
import AdmZip from "adm-zip";
// var AdmZip = require("adm-zip");
import path from "node:path";
import { filterAndSearch } from "../data/filter";
import { PrismaClient } from "@prisma/client";
import fs from "node:fs";
import { stringify } from "csv-stringify";
import { exec } from "node:child_process";
import { xlsxGenerator } from "./xlsx";
import { requestForFullData } from "../data/fetcher";
import { rimraf } from "rimraf";
const archiveRouter = Router();

const prisma = new PrismaClient();

export function deleteFolderRecursive(folderPath: string) {
  if (fs.existsSync(folderPath)) {
    fs.readdirSync(folderPath).forEach((file) => {
      const curPath = path.join(folderPath, file);
      if (fs.lstatSync(curPath).isDirectory()) {
        // Recursive call for subdirectories
        deleteFolderRecursive(curPath);
      } else {
        // Delete file
        fs.unlinkSync(curPath);
      }
    });
    // Delete the empty directory once all files and subdirectories have been deleted
    fs.rmdirSync(folderPath);
  }
}

const folderToDelete = 'path/to/folder';
deleteFolderRecursive(folderToDelete);


export const csvGenerator = async (newResponse: any, zipFolderName: string) => {
  const filename = crypto.randomUUID();
  const columns: string[] = [
    "data-id",
    "Latitude",
    "Longitude",
    "Accuracy",
    "Land Cover",
    "Description",
    "Email",
    "Water Source",
    "Crop Intensity",
    "Primary Season",
    "Primary Crop",
    "Secondary Season",
    "Secondary Crop",
    // "Livestock",
    "Cropping Pattern",
    "Crop Growth Stage",
    "Remarks",
    "Sample Size 1",
    "Sample Size 2",
    "Biomass Weight",
    "Cultivar",
    "Sow Date",
    "Harvest Date",
  ];
  const writableStream = fs.createWriteStream(
    path.join(__dirname, zipFolderName, filename + ".csv")
  );
  const stringifier = stringify({ header: true, columns: columns });
  //   let newResponse = await requestForData(req);
  for (let a of newResponse.data) {
    let row = {
      "data-id": a.id.toString(),
      Latitude: a.latitude.toString(),
      Longitude: a.longitude.toString(),
      Accuracy: a.accuracy.toString(),
      "Land Cover": a.landCover,
      Description: a.description,
      Email: a.user.email,
      "Sample Size 1": a.CCEdata[0]?.sampleSize_1,
      "Sample Size 2": a.CCEdata[0]?.sampleSize_1,
      "Biomass Weight": a.CCEdata[0]?.biomassWeight,
      Cultivar: a.CCEdata[0]?.cultivar,
      "Sow Date": a.CCEdata[0]?.sowDate.toDateString(),
      "Harvest Date": a.CCEdata[0]?.harvestDate.toDateString(),
      "Water Source": a.cropInformation[0]?.waterSource,
      "Crop Intensity": a.cropInformation[0]?.cropIntensity,
      "Primary Season": a.cropInformation[0]?.primarySeason,
      "Primary Crop": a.cropInformation[0]?.primaryCrop,
      "Secondary Season": a.cropInformation[0]?.secondarySeason,
      "Secondary Crop": a.cropInformation[0]?.secondaryCrop,
      // Livestock: a.cropInformation[0]?.livestock,
      "Cropping Pattern": a.cropInformation[0]?.croppingPattern,
      "Crop Growth Stage": a.cropInformation[0]?.cropGrowthStage,
      Remarks: a.cropInformation[0]?.remarks,
    };
    stringifier.write(row);
  }
  await stringifier.pipe(writableStream);
  await stringifier.end();

  await new Promise((resolve, reject) => {
    writableStream.on('finish', resolve);
    writableStream.on('error', reject);
  });

  return filename + ".csv";
};


enum Mode {
  csv,
  xlsx
}

const archiveMaker = async (req: Request, res: Response, mode: Mode) => {
  try {
    const newResponse = await requestForFullData(req);
    const zipFolderName = crypto.randomUUID();
    const zipFileName = zipFolderName;
    fs.mkdirSync(path.join(__dirname, zipFolderName));
    let csvfilename = "";
    let xlsxfilename = "";
    if (mode == Mode.csv) csvfilename = await csvGenerator(newResponse, zipFolderName);
    else if (mode == Mode.xlsx) xlsxfilename = await xlsxGenerator(newResponse, zipFolderName);
    // fs.copyFileSync(path.join(__dirname, csvfilename), path.join(__dirname, zipFolderName, csvfilename));
    for (let value of newResponse.data) {
      if (value.images.length > 0)
        fs.mkdirSync(
          path.join(__dirname, zipFolderName, `data-entry-${value.id}`)
        );
      for (let imageFileName of value.images) {
        fs.copyFileSync(
          path.join(
            __dirname,
            "..",
            "..",
            "..",
            "..",
            "savedImages",
            imageFileName.fileName
          ),
          path.join(
            __dirname,
            zipFolderName,
            `data-entry-${imageFileName.dataId}`,
            `data-${imageFileName.dataId}-image-${imageFileName.id}.jpg`
          )
        );
      }
    }
    const sourceFolderPath = path.join(__dirname, zipFolderName);
    const destinationZipPath = path.join(__dirname, zipFileName);
    const pythonScriptPath = path.join(__dirname, 'archiver.py');
    exec(
      `python ${pythonScriptPath} "${sourceFolderPath}" "${destinationZipPath}"`,
      function (err, stdout, stderr) {
        console.log(err, stdout, stderr);
        if (err) {
          console.error("Failed to create zip:", err);
          res.status(500).send("Failed to convert to zip");
          return;
        }

        // If 7z command executed successfully, stream the zip file to the response
        const filePath = destinationZipPath + ".zip";
        const fileStream = fs.createReadStream(filePath);
        fileStream.pipe(res);
        // fileStream.close();
        // Schedule deletion of temporary directory after some time
        const timerId = setInterval(() => {
          if (!fileStream.closed) return;
          console.log("deleted " + zipFolderName);
          rimraf.sync(sourceFolderPath);
          rimraf.sync(sourceFolderPath + ".zip")
          clearInterval(timerId)
        }, 3000);
      }
    );

  }
  catch (e) {
    console.log("error", e)
    res.send({
      success: false,
    })
  }
}


archiveRouter.post("/", async (req, res) => {
  await archiveMaker(req, res, Mode.csv);
});

archiveRouter.post("/xlsx", async (req, res) => {
  await archiveMaker(req, res, Mode.xlsx);
})


export default archiveRouter;
```

xlsx.ts, the functionalities to generate the .xlsx files

```
import path from "node:path";
import writeXlsxFile from "write-excel-file/node";
const HEADER_ROW = [
  {
    value: "data-id",
    fontWeight: "bold",
  },
  {
    value: "Latitude",
    fontWeight: "bold",
  },
  {
    value: "Longitude",
    fontWeight: "bold",
  },
  {
    value: "Accuracy",
    fontWeight: "bold",
  },
  {
    value: "Land Cover Type",
    fontWeight: "bold",
  },
  {
    value: "Description",
    fontWeight: "bold",
  },
  {
    value: "Water Source",
    fontWeight: "bold",
  },
  {
    value: "Crop Growth Stage",
    fontWeight: "bold",
  },
  {
    value: "Crop Intensity",
    fontWeight: "bold",
  },
  // {
  //   value: "Livestock",
  //   fontWeight: "bold",
  // },
  {
    value: "Cropping Pattern",
    fontWeight: "bold",
  },
  {
    value: "Primary Crop",
    fontWeight: "bold",
  },
  {
    value: "Primary Season",
    fontWeight: "bold",
  },
  {
    value: "Remarks",
    fontWeight: "bold",
  },
  {
    value: "Secondary Crop",
    fontWeight: "bold",
  },
  {
    value: "Secondary Season",
    fontWeight: "bold",
  },
  {
    value: "Biomass Weight",
    fontWeight: "bold",
  },
  {
    value: "Cultivar",
    fontWeight: "bold",
  },
  {
    value: "Grain Weight",
    fontWeight: "bold",
  },
  {
    value: "Sample Size 1",
    fontWeight: "bold",
  },
  {
    value: "Sample Size 2",
    fontWeight: "bold",
  },
  {
    value: "Harvest Date",
    fontWeight: "bold",
  },
  {
    value: "Sow Date",
    fontWeight: "bold",
  },
  {
    value: "User Email",
    fontWeight: "bold",
  },
];

export const xlsxGenerator = async (
  newResponse: any,
  zipFolderName: string
) => {
  const data = newResponse.data.map((value: any) => {
    const cropInfo =
      value.cropInformation.length > 0 ? value.cropInformation[0] : null;
    const cceData = value.CCEdata.length > 0 ? value.CCEdata[0] : null;

    return [
      { type: String, value: value.id.toString() || " " },
      { type: String, value: value.latitude.toString() || " " },
      { type: String, value: value.longitude.toString() || " " },
      { type: String, value: value.accuracy.toString() || " " },
      { type: String, value: value.landCover || "" },
      { type: String, value: value.description || "" },
      { type: String, value: cropInfo?.waterSource || "" },
      { type: String, value: cropInfo?.cropGrowthStage || "" },
      { type: String, value: cropInfo?.cropIntensity || "" },
      // { type: String, value: cropInfo?.livestock || "" },
      { type: String, value: cropInfo?.croppingPattern || "" },
      { type: String, value: cropInfo?.primaryCrop || "" },
      { type: String, value: cropInfo?.primarySeason || "" },
      { type: String, value: cropInfo?.remarks || "" },
      { type: String, value: cropInfo?.secondaryCrop || "" },
      { type: String, value: cropInfo?.secondarySeason || "" },
      { type: String, value: cceData?.biomassWeight.toString() || "" },
      { type: String, value: cceData?.cultivar || "" },
      { type: String, value: cceData?.grainWeight.toString() || "" },
      { type: String, value: cceData?.sampleSize_1.toString() || "" },
      { type: String, value: cceData?.sampleSize_2.toString() || "" },
      {
        type: Date || null,
        value: cceData?.harvestDate ? cceData?.harvestDate : "",
        format: "dd/mm/yy",
      },
      {
        type: Date || null,
        value: cceData?.sowDate ? cceData?.sowDate : "",
        format: "dd/mm/yy",
      },
      // { type: String, value: cceData?.dataId || "" },
      // { type: String, value: cceData?.id || "" },
      { type: String, value: value.user.email || "" },
    ];
  });
  const filename = crypto.randomUUID() + ".xlsx";

  const file = await writeXlsxFile([HEADER_ROW, ...data], {
    filePath: path.join(__dirname, zipFolderName, filename),
  });
  return filename;
};
```

`data` folder

fetcher.ts

```
import { PrismaClient } from "@prisma/client";
import { filterAndSearch } from "./filter";
import { Request } from "express";

const prisma = new PrismaClient();

// let data = {

// };

// async function debouncedData(){
//   setTimeout(() => {

//   }, 500)
// }

// DEBOUNCING HAS to be implemented to reduce the bandwidth of the database usage

export async function debouncer(pageNo: string, entries: string) {
  return prisma.data.findMany({
    skip: (parseInt(pageNo) - 1) * parseInt(entries),
    take: parseInt(entries),
    include: {
      cropInformation: {
        take: 10,
      },
      CCEdata: {
        take: 10,
      },
      images: {
        take: 10,
      },
      user: {
        include: {
          _count: true,
        },
      },
    },
  });
}
export const requestForFullData = async (req: Request) => {
  const body: {
    pageNo: string;
    entries: string;
    latitude: number | null;
    longitude: number | null;
    accuracy: number | null;
    landCover: string | null;
    description: string | null;
    email: string | null;
    sampleSize_1: number | null;
    sampleSize_2: number | null;
    biomassWeight: number | null;
    cultivar: string | null;
    sowDate: string | null;
    harvestDate: string | null;
    waterSource: string | null;
    cropIntensity: string | null;
    primarySeason: string | null;
    primaryCrop: string | null;
    secondarySeason: string | null;
    secondaryCrop: string | null;
    livestock: string | null;
    croppingPattern: string | null;
    cropGrowthStage: string | null;
    remarks: string | null;
  } = req.body;

  const pageNo = body.pageNo;
  const entries = body.entries;
  console.log("Latitude was sent as ", body.latitude);
  try {
    const response = await prisma.data.findMany({
      include: {
        cropInformation: {
          take: 10,
        },
        CCEdata: {
          take: 10,
        },
        images: {
          take: 10,
        },
        user: {
          include: {
            _count: true,
          },
        },
      },
    });
    const newResponse = response.filter((value) => {
      return (
        filterAndSearch(body.latitude, value.latitude) &&
        filterAndSearch(body.longitude, value.longitude) &&
        filterAndSearch(body.accuracy, value.accuracy) &&
        filterAndSearch(body.landCover, value.landCover) &&
        filterAndSearch(body.description, value.description) &&
        filterAndSearch(body.email, value.user.email) &&
        filterAndSearch(body.sampleSize_1, value.CCEdata[0]?.sampleSize_1) &&
        filterAndSearch(body.sampleSize_2, value.CCEdata[0]?.sampleSize_2) &&
        filterAndSearch(body.biomassWeight, value.CCEdata[0]?.biomassWeight) &&
        filterAndSearch(body.cultivar, value.CCEdata[0]?.cultivar) &&
        filterAndSearch(body.sowDate, value.CCEdata[0]?.sowDate) &&
        filterAndSearch(body.harvestDate, value.CCEdata[0]?.harvestDate) &&
        filterAndSearch(
          body.waterSource,
          value.cropInformation[0]?.waterSource
        ) &&
        filterAndSearch(
          body.cropIntensity,
          value.cropInformation[0]?.cropIntensity
        ) &&
        filterAndSearch(
          body.primarySeason,
          value.cropInformation[0]?.primarySeason
        ) &&
        filterAndSearch(
          body.primaryCrop,
          value.cropInformation[0]?.primaryCrop
        ) &&
        filterAndSearch(
          body.secondarySeason,
          value.cropInformation[0]?.secondarySeason
        ) &&
        filterAndSearch(
          body.secondaryCrop,
          value.cropInformation[0]?.secondaryCrop
        ) &&
        // filterAndSearch(body.livestock, value.cropInformation[0]?.livestock) &&
        filterAndSearch(
          body.croppingPattern,
          value.cropInformation[0]?.croppingPattern
        ) &&
        filterAndSearch(
          body.cropGrowthStage,
          value.cropInformation[0]?.cropGrowthStage
        ) &&
        filterAndSearch(body.remarks, value.cropInformation[0]?.remarks)
      );
    });
    const count = await prisma.data.count();
    console.log(count, "is the count");
    return {
      data: newResponse,
      count: count,
    };
  } catch (e) {
    console.log(e);
    return {
      data: [],
    };
  }
};
```

contains the functions to parse all the data from the request object, and make it accessible.

filter.ts

```

export function filterAndSearch(keyword:any, originalString:any){
    if(keyword && typeof keyword != 'string'){
        keyword = keyword.toString()
    }
    if(originalString && typeof originalString != 'string'){
        originalString = originalString.toString()
    }

    if((keyword != null && keyword != '') && originalString == undefined) return false;

    if(keyword && (originalString.startsWith(keyword.toString()) || originalString.includes(keyword))){
        return true
      }
      if(keyword == undefined || keyword == ""){
        return true
      }
      return false
}
```

contains code to help filter the data, based on the keyword for a single field

index.ts, has all the routes for deletion, and getting data for the frontend react app, and also contains types which are used by other files.

```
import { Router } from "express";
import { PrismaClient } from "@prisma/client";
import path from "node:path";
import { adminAuthMiddleware } from "../admin";
import { filterAndSearch } from "./filter";
import { debouncer } from "./fetcher";
import { authMiddleware } from "../user";
import fs from "node:fs";

const prisma = new PrismaClient();
const dataRouter = Router();
export interface imgType {
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
  harvestDate: Date;
  id: number;
  sampleSize_1: number;
  sampleSize_2: number;
  sowDate: Date;
}
export type Data = {
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
  images: imgType[];
};
dataRouter.post("", adminAuthMiddleware, async (req, res) => {
  const body: {
    pageNo: string;
    entries: string;
    latitude: number | null;
    longitude: number | null;
    accuracy: number | null;
    landCover: string | null;
    description: string | null;
    email: string | null;
    sampleSize_1: number | null;
    sampleSize_2: number | null;
    biomassWeight: number | null;
    cultivar: string | null;
    sowDate: string | null;
    harvestDate: string | null;
    waterSource: string | null;
    cropIntensity: string | null;
    primarySeason: string | null;
    primaryCrop: string | null;
    secondarySeason: string | null;
    secondaryCrop: string | null;
    // livestock: string | null;
    croppingPattern: string | null;
    cropGrowthStage: string | null;
    remarks: string | null;
  } = req.body;

  const pageNo = body.pageNo;
  const entries = body.entries;
  console.log("Latitude was sent as ", body.latitude);
  try {
    const response = await debouncer(pageNo, entries);
    const newResponse = response.filter((value) => {
      return (
        filterAndSearch(body.latitude, value.latitude) &&
        filterAndSearch(body.longitude, value.longitude) &&
        filterAndSearch(body.accuracy, value.accuracy) &&
        filterAndSearch(body.landCover, value.landCover) &&
        filterAndSearch(body.description, value.description) &&
        filterAndSearch(body.email, value.user.email) &&
        filterAndSearch(body.sampleSize_1, value.CCEdata[0]?.sampleSize_1) &&
        filterAndSearch(body.sampleSize_2, value.CCEdata[0]?.sampleSize_2) &&
        filterAndSearch(body.biomassWeight, value.CCEdata[0]?.biomassWeight) &&
        filterAndSearch(body.cultivar, value.CCEdata[0]?.cultivar) &&
        filterAndSearch(body.sowDate, value.CCEdata[0]?.sowDate) &&
        filterAndSearch(body.harvestDate, value.CCEdata[0]?.harvestDate) &&
        filterAndSearch(
          body.waterSource,
          value.cropInformation[0]?.waterSource
        ) &&
        filterAndSearch(
          body.cropIntensity,
          value.cropInformation[0]?.cropIntensity
        ) &&
        filterAndSearch(
          body.primarySeason,
          value.cropInformation[0]?.primarySeason
        ) &&
        filterAndSearch(
          body.primaryCrop,
          value.cropInformation[0]?.primaryCrop
        ) &&
        filterAndSearch(
          body.secondarySeason,
          value.cropInformation[0]?.secondarySeason
        ) &&
        filterAndSearch(
          body.secondaryCrop,
          value.cropInformation[0]?.secondaryCrop
        ) &&
        // filterAndSearch(body.livestock, value.cropInformation[0]?.livestock) &&
        filterAndSearch(
          body.croppingPattern,
          value.cropInformation[0]?.croppingPattern
        ) &&
        filterAndSearch(
          body.cropGrowthStage,
          value.cropInformation[0]?.cropGrowthStage
        ) &&
        filterAndSearch(body.remarks, value.cropInformation[0]?.remarks)
      );
    });
    const count = await prisma.data.count();
    console.log(count, "is the count");
    res.json({ response: newResponse, count: count });
    // res.json(newResponse);
  } catch (error) {
    res.json({ message: "Failed" });
    console.log(error);
  }
});

dataRouter.post("/deletemany", adminAuthMiddleware, async (req, res) => {
  const body: {
    dataId: number[];
  } = req.body;
  const dataIdArr = body.dataId;
  console.log(dataIdArr, "is the arraay we got");
  try {
    const response = await prisma.$transaction(async (pr) => {
      for (let a of dataIdArr) {
        const images = await pr.images.findMany({
          where: {
            dataId: a,
          },
        });
        await pr.images.deleteMany({
          where: {
            dataId: a,
          },
        });
        await pr.cCE.deleteMany({
          where: {
            dataId: a,
          },
        });
        await pr.cropInformation.deleteMany({
          where: {
            dataId: a,
          },
        });
        await pr.integrity.deleteMany({
          where: {
            dataId: a,
          }
        })
        await pr.data.deleteMany({
          where: {
            id: a,
          },
        });
        for (let image of images) {
          fs.unlinkSync(path.join(__dirname, "..", "..", "..", "..", "savedImages", image.fileName))
          fs.unlinkSync(path.join(__dirname, "..", "..", "..", "..", "savedImages", image.fileName + "_original"))
        }
      }
    });
    res.json({ message: "okay" });
  } catch (e) {
    res.json({
      success: false,
    });
    console.log(e);
  }
});

dataRouter.post("/deleteAll", adminAuthMiddleware, async (req, res) => {
  try {
    const images = await prisma.images.findMany({});
    await prisma.$transaction(async (prisma) => {
      const response1 = await prisma.cCE.deleteMany({});
      const response2 = await prisma.cropInformation.deleteMany({});
      const response3 = await prisma.images.deleteMany({});
      const response4 = await prisma.integrity.deleteMany({});
      const response = await prisma.data.deleteMany({});
      res.send("done");
    });
    for (let image of images) {
      fs.unlinkSync(path.join(__dirname, "..", "..", "..", "..", "savedImages", image.fileName))
      fs.unlinkSync(path.join(__dirname, "..", "..", "..", "..", "savedImages", image.fileName + "_original"))
    }
  } catch (e) {
    console.log(e);
    res.status(400).json({
      success: false,
    });
  }
});

// dataRouter.post("/cce/:id", async (req, res) => {
//     console.log(req.params.id)
// })

dataRouter.post("/:id", adminAuthMiddleware, async (req, res) => {
  console.log(req.params.id);
  let id = parseInt(req.params.id);
  if (!id) {
    id = 16;
  }
  try {
    const response = await prisma.data.findFirst({
      // take:10,
      where: {
        id: id,
      },
      include: {
        cropInformation: {
          take: 10,
        },
        CCEdata: {
          take: 10,
        },
        images: {
          take: 10,
        },
        user: {
          select: {
            email: true,
          },
        },
      },
    });
    const count = await prisma.data.count();
    console.log(count, "is the count");
    res.json({ ...response, count: count });
  } catch (error) {
    res.json({
      message: "failed",
    });
    console.log(error);
  }
});

dataRouter.get("/", async (req, res) => {
  res.send("hello");
});

dataRouter.get("/image/:filename", (req, res) => {
  try {
    const imagePath = path.join(
      __dirname,
      "..",
      "..",
      "..",
      "..",
      "savedImages",
      req.params.filename
    );
    // const image = fs.readFileSync(imagePath);
    res.sendFile(imagePath);
  } catch (error) {
    res.json({ message: "failed" });
    console.log(error);
  }
});



export default dataRouter;
```

`file` folder, it is for genreation of .xlsx, .csv, and .shp files.

index.ts

```
import { Router } from "express";
import { adminAuthMiddleware } from "../admin";
import { filterAndSearch } from "../data/filter";
import { PrismaClient } from "@prisma/client";
import fs from "node:fs";
import { stringify } from "csv-stringify";
import path from "node:path";
import xlsxFileDataHandler from "./xlsx";
import { requestForFullData } from "../data/fetcher";
import shpFileRouter from "./shp";
import { rimraf } from "rimraf";
const fileDataRouter = Router();
const prisma = new PrismaClient();
const columns: string[] = [
  "data-id",
  "Latitude",
  "Longitude",
  "Accuracy",
  "Land Cover",
  "Description",
  "Email",
  "Water Source",
  "Crop Intensity",
  "Primary Season",
  "Primary Crop",
  "Secondary Season",
  "Secondary Crop",
  // "Livestock",
  "Cropping Pattern",
  "Crop Growth Stage",
  "Remarks",
  "Sample Size 1",
  "Sample Size 2",
  "Biomass Weight",
  "Cultivar",
  "Sow Date",
  "Harvest Date",
];


fileDataRouter.post("/", adminAuthMiddleware, async (req, res) => {
  const filename = crypto.randomUUID();
  console.log(req.body, " is the whole body though")
  fs.open(path.join(__dirname, filename + ".csv"), "w", (error) => {
    console.log(error);
  });
  const writableStream = fs.createWriteStream(path.join(__dirname, filename + ".csv"));
  const stringifier = stringify({ header: true, columns: columns });
  try {
    const newResponse = await requestForFullData(req)

    // const count = await prisma.data.count();
    // console.log(count, 'is the count')
    // res.json({response: newResponse, count:count});
    // res.json(newResponse);
    for (let a of newResponse.data) {
      let row = {
        "data-id": a.id.toString(),
        "Latitude": a.latitude.toString(),
        "Longitude": a.longitude.toString(),
        "Accuracy": a.accuracy.toString(),
        "Land Cover": a.landCover,
        "Description": a.description,
        "Email": a.user.email,
        "Sample Size 1": a.CCEdata[0]?.sampleSize_1,
        "Sample Size 2": a.CCEdata[0]?.sampleSize_1,
        "Biomass Weight": a.CCEdata[0]?.biomassWeight,
        "Cultivar": a.CCEdata[0]?.cultivar,
        "Sow Date": a.CCEdata[0]?.sowDate.toDateString(),
        "Harvest Date": a.CCEdata[0]?.harvestDate.toDateString(),
        "Water Source": a.cropInformation[0]?.waterSource,
        "Crop Intensity": a.cropInformation[0]?.cropIntensity,
        "Primary Season": a.cropInformation[0]?.primarySeason,
        "Primary Crop": a.cropInformation[0]?.primaryCrop,
        "Secondary Season": a.cropInformation[0]?.secondarySeason,
        "Secondary Crop": a.cropInformation[0]?.secondaryCrop,
        // "Livestock": a.cropInformation[0]?.livestock,
        "Cropping Pattern": a.cropInformation[0]?.croppingPattern,
        "Crop Growth Stage": a.cropInformation[0]?.cropGrowthStage,
        "Remarks": a.cropInformation[0]?.remarks
      }
      stringifier.write(row);
    }
    stringifier.pipe(writableStream);
    const filePath = path.join(__dirname, filename + ".csv"); // Provide the path to your file
    const fileStream = fs.createReadStream(filePath);
    fileStream.pipe(res);

    const timerid = setInterval(() => {
      if (fileStream.closed) {
        rimraf.sync(path.join(__dirname, filename + ".csv"));
        clearInterval(timerid)
      }
    }, 3000)
  } catch (error) {
    res.json({ message: "Failed" });
    console.log(error);
  }
  finally {
    // fs.unlinkSync(path.join(__dirname, filename+".csv"));
  }
});


fileDataRouter.use("/xlsx", xlsxFileDataHandler)

fileDataRouter.use("/shp", shpFileRouter);

export default fileDataRouter;

```

main.py, this process is spawned to create .shp files using the geopandas library

```
import sys
import os
import geopandas as gpd
from shapely.geometry import Point
import pandas as pd

def convert_csv_to_shapefile(csv_filename, target_folder):
    # Read CSV file into a pandas DataFrame
    df = pd.read_csv(csv_filename)

    # Convert DataFrame to a GeoDataFrame
    geometry = [Point(lon, lat) for lat, lon in zip(df['Latitude'], df['Longitude'])]
    gdf = gpd.GeoDataFrame(df, geometry=geometry)

    # Manually specify the CRS (coordinate reference system)
    # Adjust the EPSG code according to your data's spatial reference
    crs = {'init': 'epsg:4326'}  # WGS84

    # Set the CRS for the GeoDataFrame
    gdf.crs = crs

    # Save the GeoDataFrame to a Shapefile in the target folder
    output_shapefile = os.path.join(target_folder, 'output.shp')
    gdf.to_file(output_shapefile)

if __name__ == "__main__":
    # Ensure both CSV filename and target folder name are provided
    if len(sys.argv) != 3:
        print("Usage: python convert_csv_to_shapefile.py <csv_filename> <target_folder>")
        sys.exit(1)

    # Extract command line arguments
    csv_filename = sys.argv[1]
    target_folder = sys.argv[2]

    # Create the target folder if it doesn't exist
    if not os.path.exists(target_folder):
        os.makedirs(target_folder)

    # Convert CSV to Shapefile
    convert_csv_to_shapefile(csv_filename, target_folder)

```

shp.ts

```
import { Router } from "express";
import { requestForFullData } from "../data/fetcher";
// import { csvGenerator } from "../archiveGenerator";
import fs from "node:fs"
import { exec, spawn } from "node:child_process"
import { stringify } from "csv-stringify";
// const {  } = require('child_process');
import path from "node:path";
import util from "util"
import { deleteFolderRecursive } from "../archiveGenerator";
import { rimraf } from "rimraf";
const execPromise = util.promisify(exec);
const spawnPromise = util.promisify(spawn);


const csvGenerator = async (newResponse: any, zipFolderName: string) => {
    const filename = crypto.randomUUID();
    const columns: string[] = [
        "data-id",
        "Latitude",
        "Longitude",
        "Accuracy",
        "Land Cover",
        "Description",
        "Email",
        "Water Source",
        "Crop Intensity",
        "Primary Season",
        "Primary Crop",
        "Secondary Season",
        "Secondary Crop",
        // "Livestock",
        "Cropping Pattern",
        "Crop Growth Stage",
        "Remarks",
        "Sample Size 1",
        "Sample Size 2",
        "Biomass Weight",
        "Cultivar",
        "Sow Date",
        "Harvest Date",
    ];
    const writableStream = fs.createWriteStream(
        path.join(__dirname, zipFolderName, filename + ".csv")
    );
    const stringifier = stringify({ header: true, columns: columns });
    //   let newResponse = await requestForData(req);
    for (let a of newResponse.data) {
        let row = {
            "data-id": a.id.toString(),
            Latitude: a.latitude.toString(),
            Longitude: a.longitude.toString(),
            Accuracy: a.accuracy.toString(),
            "Land Cover": a.landCover,
            Description: a.description,
            Email: a.user.email,
            "Sample Size 1": a.CCEdata[0]?.sampleSize_1,
            "Sample Size 2": a.CCEdata[0]?.sampleSize_1,
            "Biomass Weight": a.CCEdata[0]?.biomassWeight,
            Cultivar: a.CCEdata[0]?.cultivar,
            "Sow Date": a.CCEdata[0]?.sowDate.toDateString(),
            "Harvest Date": a.CCEdata[0]?.harvestDate.toDateString(),
            "Water Source": a.cropInformation[0]?.waterSource,
            "Crop Intensity": a.cropInformation[0]?.cropIntensity,
            "Primary Season": a.cropInformation[0]?.primarySeason,
            "Primary Crop": a.cropInformation[0]?.primaryCrop,
            "Secondary Season": a.cropInformation[0]?.secondarySeason,
            "Secondary Crop": a.cropInformation[0]?.secondaryCrop,
            // Livestock: a.cropInformation[0]?.livestock,
            "Cropping Pattern": a.cropInformation[0]?.croppingPattern,
            "Crop Growth Stage": a.cropInformation[0]?.cropGrowthStage,
            Remarks: a.cropInformation[0]?.remarks,
        };
        stringifier.write(row);
    }
    await stringifier.pipe(writableStream);
    return filename + ".csv";
};

function convertCsvToShapefile(csvFilename: string, targetFolder: string) {
    return new Promise((resolve, reject) => {
        const pythonScriptPath = path.join(__dirname, 'main.py');
        const pythonProcess = spawn('python', [pythonScriptPath, csvFilename, targetFolder]);
        pythonProcess.stdout.on('data', (data: string) => {
            console.log(`stdout: ${data}`);
        });

        pythonProcess.stderr.on('data', (data: string) => {
            console.error(`stderr: ${data}`);
        });
        pythonProcess.on('close', (code: number) => {
            if (code === 0) {
                resolve('Conversion successful');
            } else {
                reject(`Conversion failed with code ${code}`);
            }
        });
    });
}
// const csvFilename = 'your_csv_file.csv';
// const targetFolder = 'target_folder';

// convertCsvToShapefile(csvFilename, targetFolder)
//     .then((result) => {
//         console.log(result);
//     })
//     .catch((error) => {
//         console.error(error);
//     });


const shpFileRouter = Router();


shpFileRouter.post("", async (req, res) => {
    try {

        const newResponse = await requestForFullData(req)
        const zipFolderName = crypto.randomUUID();
        const zipFileName = zipFolderName
        fs.mkdirSync(path.join(__dirname, zipFolderName));
        const csv = await csvGenerator(newResponse, zipFolderName);
        if (csv == undefined) return;
        const csvFileName = path.join(__dirname, zipFolderName, csv);
        await convertCsvToShapefile(csvFileName, path.join(__dirname, zipFolderName));
        console.log("cst created")
        try {
            // linux
            if (process.platform == "linux") {
                exec(
                    `cd ${path.join(
                        __dirname,
                        zipFolderName
                    )} && zip -r ${zipFileName} . && cd ..`,
                    function (err, stdout, stderr) {
                        console.log(err, stdout, stderr);
                        const filePath = path.join(__dirname, zipFolderName, zipFileName);
                        const fileStream = fs.createReadStream(filePath);
                        fileStream.pipe(res);
                        fileStream.close()
                        if (err) res.send("Failed to convert to zip")
                        setTimeout(() => {
                            console.log("deleted " + zipFolderName)
                            fs.rmSync(path.join(__dirname, zipFolderName), {
                                recursive: true,
                                force: true
                            });
                        }, 200000);
                    }
                );
            }
            else if (process.platform == 'win32') {
                const sourceFolderPath = path.join(__dirname, zipFolderName);
                const destinationZipPath = path.join(__dirname, zipFileName);
                const scriptPath = path.join(__dirname, "..", "archiveGenerator", "archiver.py")
                exec(
                    `python ${scriptPath} "${destinationZipPath}" "${sourceFolderPath}"`,
                    function (err, stdout, stderr) {
                        console.log(err, stdout, stderr);
                        if (err) {
                            console.error("Failed to create zip:", err);
                            res.status(500).send("Failed to convert to zip");
                            return;
                        }

                        // If 7z command executed successfully, stream the zip file to the response
                        const filePath = destinationZipPath + ".zip";
                        const fileStream = fs.createReadStream(filePath);

                        fileStream.pipe(res);
                        // fileStream.close();
                        // Schedule deletion of temporary directory after some time
                        const timerId = setInterval(() => {
                            if (!fileStream.closed) return;
                            console.log("deleted " + zipFolderName);
                            if (process.platform == "win32") rimraf.moveRemove.sync(sourceFolderPath);
                            else rimraf.sync(sourceFolderPath);
                            rimraf.sync(sourceFolderPath + ".zip")
                            clearInterval(timerId)
                        }, 3000);
                    }
                );

            }
        }
        catch (e) {
            console.log(e, "zip file creation has failed successfully.")
        }
    }
    catch (e) {
        res.send("something")
        console.log(e)
    }
})

export default shpFileRouter;

```

xlsx.ts

```
import { Router } from "express";
import { any } from "zod";
// const writeXlsxFile = require('write-excel-file/node')
import writeXlsxFile from "write-excel-file/node";
import { Data } from "../data";
import { filterAndSearch } from "../data/filter";
import { PrismaClient } from "@prisma/client";
const xlsxFileDataHandler = Router();
import fs from "node:fs";
import { stringify } from "csv-stringify";
import path from "node:path";
import { requestForFullData } from "../data/fetcher";
import { rimraf } from "rimraf";

// await writeXlsxFile(objects, {
//   schema,
//   filePath: '/path/to/file.xlsx'
// })
const prisma = new PrismaClient();
const HEADER_ROW = [
  {
    value: "data-id",
    fontWeight: "bold",
  },
  {
    value: "Latitude",
    fontWeight: "bold",
  },
  {
    value: "Longitude",
    fontWeight: "bold",
  },
  {
    value: "Accuracy",
    fontWeight: "bold",
  },
  {
    value: "Land Cover Type",
    fontWeight: "bold",
  },
  {
    value: "Description",
    fontWeight: "bold",
  },
  {
    value: "Water Source",
    fontWeight: "bold",
  },
  {
    value: "Crop Growth Stage",
    fontWeight: "bold",
  },
  {
    value: "Crop Intensity",
    fontWeight: "bold",
  },
  // {
  //   value: "Livestock",
  //   fontWeight: "bold",
  // },
  {
    value: "Cropping Pattern",
    fontWeight: "bold",
  },
  {
    value: "Primary Crop",
    fontWeight: "bold",
  },
  {
    value: "Primary Season",
    fontWeight: "bold",
  },
  {
    value: "Remarks",
    fontWeight: "bold",
  },
  {
    value: "Secondary Crop",
    fontWeight: "bold",
  },
  {
    value: "Secondary Season",
    fontWeight: "bold",
  },
  {
    value: "Biomass Weight",
    fontWeight: "bold",
  },
  {
    value: "Cultivar",
    fontWeight: "bold",
  },
  {
    value: "Grain Weight",
    fontWeight: "bold",
  },
  {
    value: "Sample Size 1",
    fontWeight: "bold",
  },
  {
    value: "Sample Size 2",
    fontWeight: "bold",
  },
  {
    value: "Harvest Date",
    fontWeight: "bold",
  },
  {
    value: "Sow Date",
    fontWeight: "bold",
  },
  {
    value: "User Email",
    fontWeight: "bold",
  },
];

xlsxFileDataHandler.post("", async (req, res) => {
  try {
    const newResponse = await requestForFullData(req);
    const data = newResponse.data.map((value) => {
      const cropInfo =
        value.cropInformation.length > 0 ? value.cropInformation[0] : null;
      const cceData = value.CCEdata.length > 0 ? value.CCEdata[0] : null;

      return [
        { type: String, value: value.id.toString() || " " },
        { type: String, value: value.latitude.toString() || " " },
        { type: String, value: value.longitude.toString() || " " },
        { type: String, value: value.accuracy.toString() || " " },
        { type: String, value: value.landCover || "" },
        { type: String, value: value.description || "" },
        { type: String, value: cropInfo?.waterSource || "" },
        { type: String, value: cropInfo?.cropGrowthStage || "" },
        { type: String, value: cropInfo?.cropIntensity || "" },
        // { type: String, value: cropInfo?.livestock || "" },
        { type: String, value: cropInfo?.croppingPattern || "" },
        { type: String, value: cropInfo?.primaryCrop || "" },
        { type: String, value: cropInfo?.primarySeason || "" },
        { type: String, value: cropInfo?.remarks || "" },
        { type: String, value: cropInfo?.secondaryCrop || "" },
        { type: String, value: cropInfo?.secondarySeason || "" },
        { type: String, value: cceData?.biomassWeight.toString() || "" },
        { type: String, value: cceData?.cultivar || "" },
        { type: String, value: cceData?.grainWeight.toString() || "" },
        { type: String, value: cceData?.sampleSize_1.toString() || "" },
        { type: String, value: cceData?.sampleSize_2.toString() || "" },
        {
          type: Date || null,
          value: cceData?.harvestDate ? cceData?.harvestDate : "",
          format: "dd/mm/yy",
        },
        {
          type: Date || null,
          value: cceData?.sowDate ? cceData?.sowDate : "",
          format: "dd/mm/yy",
        },
        // { type: String, value: cceData?.dataId || "" },
        // { type: String, value: cceData?.id || "" },
        { type: String, value: value.user.email || "" },
      ];
    });
    const filename = crypto.randomUUID() + ".xlsx";

    // @ts-ignore
    const file = await writeXlsxFile([HEADER_ROW, ...data], {
      filePath: path.join(__dirname, filename),
    });

    const filePath = path.join(__dirname, filename); // Provide the path to your file
    const fileStream = fs.createReadStream(filePath);
    fileStream.pipe(res);

    const timerid = setInterval(() => {
      if (fileStream.closed) {
        rimraf.sync(path.join(__dirname, filename));
        clearInterval(timerid)
      }
    }, 3000);
  } catch (e) {
    res.json({
      message: "failed",
    });
    console.log(e);
  }
});

export default xlsxFileDataHandler;

```

the `sync` folder, the code to sync the data from the native-app to this backend, it uses transactions as before mentioned.

imageMetaDataSetter.ts

```
const { ExifTool } = require('exiftool-vendored');
import path from "node:path"

export async function setGPSMetadata(imagePath: string, latitude: number | undefined, longitude: number | undefined) {
  const exiftool = new ExifTool();
  if(latitude == undefined || longitude == undefined ) throw Error();
  console.log(latitude, "IS THE LAT FOR GPS", longitude, "IS THE LONG")
  // imagePath = path.join(__dirname, "..", "..", "..", "savedImages", imagePath)

  try {
    // Convert latitude and longitude to the format required by ExifTool
    const latRef = latitude >= 0 ? 'N' : 'S';
    const lonRef = longitude >= 0 ? 'E' : 'W';
    const absLat = Math.abs(latitude);
    const absLon = Math.abs(longitude);

    const latDeg = Math.floor(absLat);
    const latMin = Math.floor((absLat - latDeg) * 60);
    const latSec = ((absLat - latDeg) * 60 - latMin) * 60;

    const lonDeg = Math.floor(absLon);
    const lonMin = Math.floor((absLon - lonDeg) * 60);
    const lonSec = ((absLon - lonDeg) * 60 - lonMin) * 60;

    // Set GPS metadata
    await exiftool.write(imagePath, {
      GPSLatitudeRef: latRef,
      GPSLatitude: `${latDeg} ${latMin} ${latSec.toFixed(2)}`,
      GPSLongitudeRef: lonRef,
      GPSLongitude: `${lonDeg} ${lonMin} ${lonSec.toFixed(2)}`,
    });

    console.log(`GPS metadata set to Latitude: ${latitude}, Longitude: ${longitude}`);
  } catch (error) {
    console.error('Error setting GPS metadata:', error);
  } finally {
    // Ensure the ExifTool process is closed to prevent hanging
    await exiftool.end();
  }
}

```

index.ts

```
import singleSyncRouter from "./single"

export {singleSyncRouter}
```

single.ts

```
import { Router } from "express";
import { PrismaClient } from "@prisma/client";
import { saveBase64File } from "../../fileIntercept";
import { authMiddleware } from "../user";
import { setGPSMetadata } from "./imageMetaDataSetter";
import fs from "fs";
import { Transactions } from "../../transaction/transaction";
const prisma = new PrismaClient();
const singleSyncRouter = Router();

Transactions.getInstance();

const DeleteTimeout = 60 * 1000; // MILLISECONDS - minute

function isImagePresent(imagePath: string, directoryPath: string) {
  const fullPath = directoryPath + '/' + imagePath;
  return fs.existsSync(fullPath);
}



singleSyncRouter.post("/", authMiddleware, async (req, res) => {
  try {
    console.log(req.body);
    const body = req.body;
    console.log("THE DATA IS", body);
    let success = true;
    let result;
    // do some checks if the data is integral or not.
    if (req.userId) result = Transactions.getInstance().insertData(body, req.userId);
    console.log(result)
    if (result?.dataId) Transactions.getInstance().destructAndClean(result?.dataId);
    res.json({
      message: "succes",
      success: true,
      dataId: result?.dataId,
    });
  } catch (e) {
    console.log("error", e);
    res.json({
      success: false,
    });
  }
});

singleSyncRouter.post("/image", authMiddleware, async (req, res) => {
  try {
    const fileData = req.body.fileData;
    // also intercept the extension of the file
    // console.log("SAVED THE IMAGE CHECK POINT 1", fileData);
    const fileName = `${crypto.randomUUID()}.jpg`;
    if (!Transactions.getInstance().ifDataExists(req.body.dataId)) {
      throw Error();
    }
    const dataId = req.body.dataId;
    const result = Transactions.getInstance()?.latitudeAndLongitude(dataId);
    const latitude = result?.latitude;
    const longitude = result?.longitude;
    saveBase64File(fileData, fileName, latitude, longitude);
    Transactions.getInstance()?.syncImageUpdate(dataId, fileName);
    console.log(fileName, "dataid:", req.body.dataId);
    console.log("SAVED THE IMAGE CHECK POINT 2")
    console.log("THE DATAID", req.body.dataId)
    Transactions.getInstance()?.checkAndPush(dataId);
    console.log("SAVED THE IMAGE CHECK POINT 3");
    // const latitude: number = data?.latitude.toNumber();
    // setGPSMetadata(fileName, data?.latitude.toNumber(), data?.longitude.toNumber());
    res.json({
      success: true,
    });
    console.log("IMAGE RESPONSE WAS ", true)
  } catch (e) {
    console.log("NOT SAVED THE IMAGE CHECK POINT 4");
    console.log(e);
    res.json({
      success: false,
    });
  }
});

singleSyncRouter.post("/synced/", authMiddleware, async (req, res) => {
  console.log("helloweas")
  try {
    const userId = req.userId;
    const response = await prisma.user.findFirst({
      where: {
        id: userId
      }
    })
    const response2 = await prisma.data.findMany({
      where: {
        userId: userId
      }
    })
    console.log("SYNCED Data request it is ", response?.synced);
    res.json({
      synced: response?.synced,
      success: true
    })
  }
  catch (error) {
    console.log(error, " IS THE ERROR")
  }
})

export default singleSyncRouter;
```

the single here signifies, it is happening for each entries, each image a single request, each data entry a single request, rather than a combined request, which does not seem that favourable in this case, but can be made and then split to handle into chunks.

the `user` folder

like the admin folder, this has the authentication for normal users on the native-app only.

```
import { PrismaClient } from "@prisma/client";
import { NextFunction, Request, Response, Router } from "express";
import { sign, verify } from "jsonwebtoken";
const userRouter = Router();

const prisma = new PrismaClient();
// @ts-ignore
export const JWT_SECRET: string = process.env.JWT_SECRET;
// very important
declare global {
  namespace Express {
    interface Request {
      userId?: number;
    }
  }
}

export const authMiddleware = async (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  const authHeader: string | undefined = req.headers.authorization;
  const jwt = authHeader?.split(" ")[1];
  if (jwt)
    try {
      const data = await verify(jwt, JWT_SECRET);
      // @ts-ignore
      const id = data.id;
      console.log(data, "user id");
      // next();
      const response = await prisma.user.findUniqueOrThrow({
        where: {
          id: id,
        },
      });
      console.log(response, "user found with that jwt");
      req.userId = id;
      next();
    } catch (e) {
      res.json({
        success: false,
      });
      console.log(e);
    }
};

userRouter.post("/signup", async (req, res) => {
  const body: {
    email: string;
    password: string;
    Designation: string;
    Institute: string;
    Province: string;
    Country: string;
    username: string;
  } = req.body;

  try {
    const response = await prisma.user.create({
      data: {
        Country: body.Country,
        Designation: body.Designation,
        email: body.email,
        Institute: body.Institute,
        password: body.password,
        Province: body.Province,
        admin: false,
        synced: 0
      },
    });
    console.log(response);
    res.json({
      success: true,
    });
  } catch (e) {
    console.log(e);
    res.json({
      success: false,
    });
  }
});

userRouter.post("/login", async (req, res) => {
  const body: {
    email: string;
    password: string;
  } = req.body;
  try {
    const response = await prisma.user.findUniqueOrThrow({
      where: {
        email: body.email,
        password: body.password,
      },
    });
    const jwt = await sign({ id: response.id }, JWT_SECRET);
    res.send({ jwt, success: true, email: response.email });
  } catch (e) {
    console.log(e);
    res.send({ success: false });
  }
});
export default userRouter;

```
