---
title: Tutorial - Prepare a React single-page app (SPA) for authentication in a customer tenant
description: Learn how to prepare a React single-page app (SPA) for authentication with your Azure Active Directory (AD) for customers tenant.
services: active-directory
author: garrodonnell
manager: celestedg
ms.service: active-directory
ms.subservice: ciam
ms.topic: tutorial
ms.date: 05/23/2023
ms.author: godonnell
#Customer intent: As a dev, devops, or IT admin, I want to learn how to enable authentication in my own React single-page app
---

# Tutorial: Prepare a React single-page app (SPA) for authentication in a customer tenant

In the [previous article](./how-to-single-page-application-react-prepare-tenant.md), you registered an application and configured user flows in your Azure Active Directory (AD) for customers tenant. This tutorial demonstrates how to create a React single-page app using `npm` and create files needed for authentication and authorization.

In this tutorial you'll;

> [!div class="checklist"]
> * Create a React project in Visual Studio Code
> * Install identity and bootstrap packages
> * Configure the settings for the application

## Prerequisites

* Completion of the prerequisites and steps in [Prepare your customer tenant to authenticate users in a React single-page app (SPA)](./how-to-single-page-application-react-prepare-tenant.md).
* Although any integrated development environment (IDE) that supports React applications can be used, this tutorial uses **Visual Studio Code**. You can download it [here](https://visualstudio.microsoft.com/downloads/).
* [Node.js](https://nodejs.org/en/download/).

## Create a React project

1. Open Visual Studio Code, select **File** > **Open Folder...**. Navigate to and select the location in which to create your project.
1. Open a new terminal by selecting **Terminal** > **New Terminal**.
1. Run the following commands to create a new React project with the name *reactspalocal*, change to the new directory and start the React project. A web browser will open with the address `http://localhost:3000/` by default. The browser remains open and re-renders for every saved change.

    ```powershell
    npx create-react-app reactspalocal
    cd reactspalocal
    npm start
    ```

## Install identity and bootstrap packages

Identity related **npm** packages must be installed in the project to enable user authentication. For project styling, we'll use **Bootstrap**.

1. In the **Terminal** bar, select the **+** icon to create a new terminal. A new terminal window will open enabling the other terminal to continue running in the background.
1. If necessary, navigate to the *reactspalocal* again and enter the following commands into the terminal to install the relevant `msal` and `bootstrap` packages.

    ```powershell
    npm install @azure/msal-browser @azure/msal-react
    npm install react-bootstrap bootstrap
    ```

## Create the authentication configuration file, *authConfig.js*

1. Navigate to the *src* folder, and create a new file called *authConfig.js*.
1. Open *authConfig.js* and add the following code snippet:

    ```javascript
    /*
     * Copyright (c) Microsoft Corporation. All rights reserved.
     * Licensed under the MIT License.
     */
    
    import { LogLevel } from '@azure/msal-browser';
    
    /**
     * Configuration object to be passed to MSAL instance on creation. 
     * For a full list of MSAL.js configuration parameters, visit:
     * https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser/docs/configuration.md 
     */
    
    export const msalConfig = {
        auth: {
            clientId: 'Enter_the_Application_Id_Here', // This is the ONLY mandatory field that you need to supply.
            authority: 'https://Enter_the_Tenant_Subdomain_Here.ciamlogin.com/', // Replace the placeholder with your tenant subdomain 
            redirectUri: '/', // Points to window.location.origin. You must register this URI on Azure Portal/App Registration.
            postLogoutRedirectUri: '/', // Indicates the page to navigate after logout.
            navigateToLoginRequestUrl: false, // If "true", will navigate back to the original request location before processing the auth code response.
        },
        cache: {
            cacheLocation: 'sessionStorage', // Configures cache location. "sessionStorage" is more secure, but "localStorage" gives you SSO between tabs.
            storeAuthStateInCookie: false, // Set this to "true" if you are having issues on IE11 or Edge
        },
        system: {
            loggerOptions: {
                loggerCallback: (level, message, containsPii) => {
                    if (containsPii) {
                        return;
                    }
                    switch (level) {
                        case LogLevel.Error:
                            console.error(message);
                            return;
                        case LogLevel.Info:
                            console.info(message);
                            return;
                        case LogLevel.Verbose:
                            console.debug(message);
                            return;
                        case LogLevel.Warning:
                            console.warn(message);
                            return;
                        default:
                            return;
                    }
                },
            },
        },
    };
    
    /**
     * Scopes you add here will be prompted for user consent during sign-in.
     * By default, MSAL.js will add OIDC scopes (openid, profile, email) to any login request.
     * For more information about OIDC scopes, visit: 
     * https://docs.microsoft.com/azure/active-directory/develop/v2-permissions-and-consent#openid-connect-scopes
     */
    export const loginRequest = {
        scopes: [],
    };
    
    /**
     * An optional silentRequest object can be used to achieve silent SSO
     * between applications by providing a "login_hint" property.
     */
    // export const silentRequest = {
    //     scopes: ["openid", "profile"],
    //     loginHint: "example@domain.net"
    // };
    ```

1. Replace the following values with the values from the Azure portal:
    - Replace `Enter_the_Application_Id_Here` with the **Application (client) ID** value that was recorded earlier from the **Overview** page of the registered application.
    - In **Authority**, find `Enter_the_Tenant_Subdomain_Here` and replace it with the subdomain of your tenant. For example, if your tenant primary domain is *caseyjensen@onmicrosoft.com*, the value you should enter is *casyjensen*.

## Modify *index.js* to include the authentication provider

All parts of the app that require authentication must be wrapped in the [`MsalProvider`](/javascript/api/@azure/msal-react/#@azure-msal-react-msalprovider) component. You instantiate a [PublicClientApplication](/javascript/api/@azure/msal-browser/publicclientapplication) then pass it to `MsalProvider`.

1. In the *src* folder, open *index.js* and replace the contents of the file with the following code snippet to use the `msal` packages and bootstrap styling:

    ```javascript
    import React from 'react';
    import ReactDOM from 'react-dom/client';
    import App from './App';
    import { PublicClientApplication, EventType } from '@azure/msal-browser';
    import { msalConfig } from './authConfig';
    
    import 'bootstrap/dist/css/bootstrap.min.css';
    import './styles/index.css';
    
    /**
     * MSAL should be instantiated outside of the component tree to prevent it from being re-instantiated on re-renders.
     * For more, visit: https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-react/docs/getting-started.md
     */
    const msalInstance = new PublicClientApplication(msalConfig);
    
    // Default to using the first account if no account is active on page load
    if (!msalInstance.getActiveAccount() && msalInstance.getAllAccounts().length > 0) {
        // Account selection logic is app dependent. Adjust as needed for different use cases.
        msalInstance.setActiveAccount(msalInstance.getActiveAccount()[0]);
    }
    
    // Listen for sign-in event and set active account
    msalInstance.addEventCallback((event) => {
        if (event.eventType === EventType.LOGIN_SUCCESS && event.payload.account) {
            const account = event.payload.account;
            msalInstance.setActiveAccount(account);
        }
    });
    
    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(
        <App instance={msalInstance}/>
    );
	```

## Next steps

> [!div class="nextstepaction"]
> [Sign in and sign out of the React SPA](./how-to-single-page-application-react-sign-in-out.md)