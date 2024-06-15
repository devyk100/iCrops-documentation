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
