---
title: Tutorial - Add sign-in and sign-out to a React single-page app (SPA) for a customer tenant
description: Learn how to configure a React single-page app (SPA) to sign in and sign out users with your Azure Active Directory (AD) for customers tenant.
services: active-directory
author: godonnell
manager: celestedg

ms.service: active-directory
ms.subservice: ciam
ms.topic: tutorial
ms.date: 05/23/2023
ms.author: godonnell

#Customer intent: As a developer I want to add sign-in and sign-out functionality to my React single-page app
---

# Tutorial: Add sign-in and sign-out to a React single-page app (SPA) for a customer tenant

In the [previous article](./how-to-single-page-application-react-prepare-app.md), you created a React single-page app (SPA) in Visual Studio Code and configured it for authentication.

In this tutorial you'll;

> [!div class="checklist"]
> * Add functional components to the application
> * Create a page layout and add the sign in and sign out experience
> * Replace the default function to render authenticated information
> * Sign in and sign out of the application using the user flow

## Prerequisites

* Completion of the prerequisites and steps in [Prepare an single-page app for authentication](./how-to-single-page-application-react-prepare-app.md).

## Add components to the application

Functional components are the building blocks of React apps, and are used to build the sign in and sign out experiences in a React SPA. 

1. Right click on *src*, select **New Folder** and call it *components*.
1. Right click on *components* and using the **New File** option, create the following files to create a structure as depicted in the following code block;
    - *PageLayout.jsx*
    - *SignInButton.jsx*
    - *SignOutButton.jsx*

    ```txt
    reactspalocal/
    ├── src/
    │   ├── components/
    │   │   ├── PageLayout.jsx
    │   │   ├── SignInButton.jsx
    │   │   └── SignOutButton.jsx
    │   └── ...
    └── ...
    ```

### Add the page layout

1. Open *PageLayout.jsx* and add the following code to render the page layout. The [useIsAuthenticated](/javascript/api/@azure/msal-react) hook returns whether or not a user is currently signed-in.

   ```javascript
   /*
    * Copyright (c) Microsoft Corporation. All rights reserved.
    * Licensed under the MIT License.
    */

   import React from "react";
   import Navbar from "react-bootstrap/Navbar";

   import { useIsAuthenticated } from "@azure/msal-react";
   import { SignInButton } from "./SignInButton";
   import { SignOutButton } from "./SignOutButton";

   /**
    * Renders the navbar component with a sign in or sign out button depending on whether or not a user is authenticated
    * @param props
    */
   export const PageLayout = (props) => {
     const isAuthenticated = useIsAuthenticated();

     return (
       <>
         <Navbar bg="primary" variant="dark" className="navbarStyle">
           <a className="navbar-brand" href="/">
             Microsoft Identity Platform
           </a>
           <div className="collapse navbar-collapse justify-content-end">
             {isAuthenticated ? <SignOutButton /> : <SignInButton />}
           </div>
         </Navbar>
         <br />
         <br />
         <h5>
           <center>
             Welcome to the Microsoft Authentication Library For Javascript -
             React SPA Tutorial
           </center>
         </h5>
         <br />
         <br />
         {props.children}
       </>
     );
   };
   ```

1. Save the file.

### Add the sign in experience

1. Open *SignInButton.jsx* and add the following code, which creates a button that signs in the user using either a pop-up or redirect. The `useMsal` hook is used to retrieve an access token to allow user sign in:

   ```javascript 
   import React from "react";
   import { useMsal } from "@azure/msal-react";
   import { loginRequest } from "../authConfig";
   import DropdownButton from "react-bootstrap/DropdownButton";
   import Dropdown from "react-bootstrap/Dropdown";

   /**
    * Renders a drop down button with child buttons for logging in with a popup or redirect
    * Note the [useMsal] package 
    */

   export const SignInButton = () => {
     const { instance } = useMsal();

     const handleLogin = (loginType) => {
       if (loginType === "popup") {
         instance.loginPopup( 
         ...loginRequest,
                redirectUri: '/redirect',
                ).catch((e) => {
           console.log(e);
         });
       } else if (loginType === "redirect") {
         instance.loginRedirect(loginRequest).catch((e) => {
           console.log(e);
         });
       }
     };
     return (
       <DropdownButton
         variant="secondary"
         className="ml-auto"
         drop="start"
         title="Sign In"
       >
         <Dropdown.Item as="button" onClick={() => handleLogin("popup")}>
           Sign in using Popup
         </Dropdown.Item>
         <Dropdown.Item as="button" onClick={() => handleLogin("redirect")}>
           Sign in using Redirect
         </Dropdown.Item>
       </DropdownButton>
     );
   };
   ```

1. Save the file.

### Add the sign out experience

1. Open *SignOutButton.jsx* and add the following code, which creates a button that signs out the user using either a pop-up or redirect.

   ```javascript 
   import React from "react";
   import { useMsal } from "@azure/msal-react";
   import DropdownButton from "react-bootstrap/DropdownButton";
   import Dropdown from "react-bootstrap/Dropdown";

   /**
    * Renders a sign out button 
    */
   export const SignOutButton = () => {
     const { instance } = useMsal();

     const handleLogout = (logoutType) => {
       if (logoutType === "popup") {
         instance.logoutPopup({
           postLogoutRedirectUri: "/",
           mainWindowRedirectUri: "/",
         });
       } else if (logoutType === "redirect") {
         instance.logoutRedirect({
           postLogoutRedirectUri: "/",
         });
       }
     };

     return (
       <DropdownButton
         variant="secondary"
         className="ml-auto"
         drop="start"
         title="Sign Out"
       >
         <Dropdown.Item as="button" onClick={() => handleLogout("popup")}>
           Sign out using Popup
         </Dropdown.Item>
         <Dropdown.Item as="button" onClick={() => handleLogout("redirect")}>
           Sign out using Redirect
         </Dropdown.Item>
       </DropdownButton>
     );
   };
   ```

## Change filename and add required imports

By default, the application runs via a JavaScript file called *App.js*. It needs to be changed to a *.jsx* file, which is an extension that allows a developer to write HTML in React.

1. Rename *App.js* to *App.jsx*.
1. Replace the existing imports with the following snippet:

   ```javascript
   import React, { useState } from 'react';

   import { PageLayout } from './components/PageLayout';
   import { loginRequest } from './authConfig';

   import { AuthenticatedTemplate, UnauthenticatedTemplate, useMsal } from '@azure/msal-react';

   import './App.css';

   import Button from 'react-bootstrap/Button';
   ```

### Replacing the default function to render authenticated information

1. Replace the default function `App()` to render authenticated information with the following code:

    ```javascript
    /**
    * If a user is authenticated the ProfileContent component above is rendered. Otherwise a message indicating a user is not authenticated is rendered.
    */
    const MainContent = () => {
        return (
            <div className="App">
                <AuthenticatedTemplate>
                    <ProfileContent />
                </AuthenticatedTemplate>
        
                <UnauthenticatedTemplate>
                    <h5>
                        <center>
                            Please sign-in to see your profile information.
                        </center>
                    </h5>
                </UnauthenticatedTemplate>
            </div>
        );
    };
        
    export default function App() {
        return (
            <PageLayout>
                <center>
                    <MainContent />
                </center>
            </PageLayout>
        );
    }
    ```

## Run your project and sign in

All the required code snippets have been added, so the application can now be tested in a web browser.

1. The application should already be running in your terminal. If not, run the following command to start your app.

    ```powershell
    npm start
    ```

1. Open a web browser and navigate to `http://localhost:3000/` if you are not automatically redirected.
1. For the purposes of this tutorial, choose the **Sign in using Popup** option.
1. After the popup window appears with the sign-in options, select the account with which to sign-in.
1. A second window may appear indicating that a code will be sent to your email address. If this happens, select **Send code**. Open the email from the sender Microsoft account team, and enter the 7-digit single-use code. Once entered, select **Sign in**.
1. For **Stay signed in**, you can select either **No** or **Yes**.
1. The app will now ask for permission to sign-in and access data. Select **Accept** to continue.

## Sign out of the application

1. To sign out of the application, select **Sign out** in the navigation bar.
1. A window appears asking which account to sign out of.
1. Upon successful sign out, a final window appears advising you to close all browser windows.

## Next steps

> [!div class="nextstepaction"]
> [Enable self-service password reset](./how-to-enable-password-reset-customers.md)