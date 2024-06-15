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
