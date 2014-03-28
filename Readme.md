WebApp-MultiTenant-OpenIDConnect-DotNet
===========================

This sample shows how to build a multi-tenant .Net MVC web application that uses OpenID Connect to sign up and sign in users from any Azure Active Directory tenant, using the ASP.Net OpenID Connect OWIN middleware and the Active Directory Authentication Library (ADAL) for .NET.

For more information about how the protocols work in this scenario and other scenarios, see the [Authentication Scenarios for Azure AD](http://msdn.microsoft.com/aad) document.

## How To Run This Sample

Getting started is simple!  To run this sample you will need:
- Visual Studio 2013
- An Internet connection
- An Azure subscription (a free trial is sufficient)

Every Azure subscription has an associated Azure Active Directory tenant. You can easily create new tenants within your subscription for testing purposes   If you don't already have an Azure subscription, you can get a free subscription by signing up at [http://wwww.windowsazure.com](http://www.windowsazure.com).  All of the Azure AD features used by this sample are available free of charge.

### Step 1:  Clone or download this repository

From your shell or command line:

`git clone git@github.com:AzureADSamples/WebApp-MultiTenant-OpenIdConnect-DotNet.git`

### Step 2:  Create a user account in your Azure Active Directory tenant

If you already have a user account in your Azure Active Directory tenant, you can skip to the next step.  This sample will not work with a Microsoft account, so if you signed in to the Azure portal with a Microsoft account and have never created a user account in your directory before, you need to do that now. You can find instructions to do that [here](http://www.cloudidentity.com/blog/2013/12/11/setting-up-an-asp-net-project-with-organizational-authentication-requires-an-organizational-account/).  If you create an account and want to use it to sign-in to the Azure portal, don't forget to add the user account as a co-administrator of your Azure subscription.

### Step 3:  Register the sample with your Azure Active Directory tenant

1. Sign in to the [Azure management portal](https://manage.windowsazure.com).
2. Click on Active Directory in the left hand nav.
3. Click the directory tenant where you wish to register the sample application.
4. Click the Applications tab.
5. In the drawer, click Add.
6. Click "Add an application my organization is developing".
7. Enter a friendly name for the application, for example "TodoListWebApp_MT", select "Web Application and/or Web API", and click next.
8. For the sign-on URL, enter the base URL for the sample, which is by default `https://localhost:44302/`.
9. For the App ID URI, enter `https://<your_tenant_domain>/TodoListWebApp_MT`, replacing `<your_tenant_domain>` with the domain of your Azure AD tenant (either in the form `<tenant_name>.onmicrosoft.com` or your own custom domain if you registered it in Azure Active Directory).
10. Click the Finish button on the lower right corner. Your application is now provisioned.
11. Click on the Configure tab of your application
12. Find "the application is multi-tenant" switch and flip it to yes. Hit the Save button from the command bar.
12. Scroll down to the bottom of the page, to the section Permissions to other applications". On the Windows Azure Active Directory row, open the Delegated Permissions combo box and select "Enable sign-on and read users' profiles". Hit Save again.

Don't close the browser yet, as we will still need to work with the portal for few more steps. 

### Step 4:  Provision a key for your app in your Azure Active Directory tenant

The new customer onboarding process implemented by the sample requires the application to perform an OAuth2 request, which in turn requires to associate a key to the app in your tenant.
 
1. Click the Configure tab of your application.
2. Find the Keys section. In the Select Duration dropdown, pick a value.
3. Hit the Save button on the bottom command bar.
4. Once the save operation terminates, the value of the key appears. Copy the key to the clipboard. **Important: this is the only opportunity you have to access the value of the key, if you don't use it now you'll have to create a new one.**

Leave the browser open to this page. 

### Step 5:  Configure the sample to use your Azure Active Directory tenant

At this point we are ready to paste into the VS project the settings that will tie it to its entry in your Azure AD tenant. 

1. Open the solution in Visual Studio 2013.
2. Open the `web.config` file.
3. Find the app key `ida:Password` and replace the value you copied in step 4.
4. Go back to the portal, find the CLIENT ID field and copy its content to the clipboard
5. Find the app key `ida:ClientId` and replace the value with the CLIENT ID from the Azure portal.

### Step 6:  [optional] Create an Azure Active Directory test tenant 

This sample shows how to take advantage of the consent model in Azure AD to make an application available to any user from any organization with a tenant in Azure AD. To see that part of the sample in action, you need to have access to user accounts from a tenant that is different from the one you used for developing the application. The simplest way of doing that is to create a new directory tenant in your Azure subscription (just navigate to the main Active Directory page in the portal and click Add) and add test users.
This step is optional as you can also use accounts from the same directory, but if you do you will not see the consent prompts as the app is already approved. 

### Step 5:  Run the sample

The sample implements two distinct tasks: the onboarding of a new customer and regular sign in & use of the application.
####  Sign up
Start the application. Click on Sign Up.
You will be presented with a form that simulates an onboarding process. Here you can choose if you want to follow the "admin consent" flow (the app gets provisioned for all the users in one organization - requires you to sign up using an administrator) or the "user consent" flow (the app gets provisioned for your user only).
Click the SignUp button. You'll be transferred to the Azure AD portal. Sign in as the user you want to use for consenting. If the user is from a tenant that is different from the one where the app was developed, you will be presented with a consent page. Click OK. You will be transported back to the app, where your registration will be finalized.
####  Sign in
Once you signed up, you can either click on the Todo tab or the sign in link to gain access to the application. Note that if you are doing this in the same session in whihc you signed up, you will automatically sign in with the same account you used for signing up. If you are signing in during a new session, you will be presented with Azure AD's credentials prompt: sign in using an account compatible with the sign up option you chose earlier (the exact same account if you used user consent, any user form the same tenant if you used admin consent). 

## How To Deploy This Sample to Azure

Coming soon.

## About The Code

<STILL WORKING ON THIS>

This sample shows how to use the OpenID Connect ASP.Net OWIN middleware to sign-in users from a single Azure AD tenant.  The middleware is initialized in the `Startup.Auth.cs` file, by passing it the Client ID of the application and the URL of the Azure AD tenant where the application is registered.  The middleware then takes care of:
- Downloading the Azure AD metadata, finding the signing keys, and finding the issuer name for the tenant.
- Processing OpenID Connect sign-in responses by validating the signature and issuer in an incoming JWT, extracting the user's claims, and putting them on ClaimsPrincipal.Current.
- Integrating with the session cookie ASP.Net OWIN middleware to establish a session for the user. 

You can trigger the middleware to send an OpenID Connect sign-in request by decorating a class or method with the `[Authorize]` attribute, or by issuing a challenge,
```C#
HttpContext.GetOwinContext().Authentication.Challenge(
	new AuthenticationProperties { RedirectUri = "/" },
	OpenIdConnectAuthenticationDefaults.AuthenticationType);
```
Similarly you can send a signout request,
```C#
HttpContext.GetOwinContext().Authentication.SignOut(
	OpenIdConnectAuthenticationDefaults.AuthenticationType,
	CookieAuthenticationDefaults.AuthenticationType);
```
When a user is signed out, they will be redirected to the `Post_Logout_Redirect_Uri` specified when the OpenID Connect middleware is initialized.

All of the OWIN middleware in this project is created as a part of the open source [Katana project](http://katanaproject.codeplex.com).  You can read more about OWIN [here](http://owin.org).
