---
# required metadata

title: Restrict access to networks with Cisco ISE | Microsoft Intune
description: Use Cisco ISE with Intune so that devices are Intune enrolled and policy-compliant before they access WiFi and VPN controlled by Cisco ISE.
keywords:
author: nbigman
manager: jeffgilb
ms.date: 06/24/2016
ms.topic: article
ms.prod:
ms.service: microsoft-intune
ms.technology:
ms.assetid: 5631bac3-921d-438e-a320-d9061d88726c

# optional metadata

#ROBOTS:
#audience:
#ms.devlang:
#ms.reviewer: muhosabe
#ms.suite: ems
#ms.tgt_pltfrm:
#ms.custom:

---

# Using Cisco ISE with Microsoft Intune
Intune integration with Cisco ISE allows you to author network policies in your ISE environment using the Intune device-enrollment and compliance state. These policies can work to ensure that access to your company network is restricted to devices that are managed by Intune and compliant with Intune policies. 

## Configuration

To enable this integration, you don’t need to do any setup in your Intune tenant. You will need to provide permissions to your Cisco ISE server to access to your Intune tenant and once that is done, the rest of the setup happens in your Cisco ISE server. This article provides you instructions on providing your ISE server with permissions to access your Intune tenant. 

### Step 1: Manage the certificates
1. In the Azure Active Directory (AAD) console, export the certificate. 

	#### Internet Explorer 11
		
    a. Run Internet Explorer as an administrator, and log in to the AAD console.
  
    b. Choose the lock icon in the address bar and choose **View certificates**
    
    c. On the **Details** tab of the certificate properties, choose **Copy to file**.

    d. In the **Certificate export wizard** welcome page, choose **Next**. 

    e. On the **Export file format** page, leave the default, **DER encoded binary x.509 (.CER)**, and choose **Next**.  

    f. On the **File to export** page, choose **Browse** to pick a location in which to save the file, and provide a file name. Though it seems like you’re picking a file to export, you’re actually naming the file to which the exported certificate will be saved. Choose **Next** &gt; **Finish**.

	#### Safari
	
	a. Log in to the AAD console.

	b. Choose the lock icon &gt;  **More information**.
    
    c. Choose **View certificate** &gt; **Details**.

    d. Choose the certificate, and then choose **Export**.  


> [!IMPORTANT]
> Check the expiration date of the certificate, as you will have to export and import a new certificate when this one expires.

	

2. From within the ISE console, import the Intune certificate (the file you exported) into the  **Trusted Certificates** store.
3. In your ISE console, go to **Administration** > **Certificates** > **System Certificates**.
4. Select the ISE certificate and then choose **Export**.
5. In a text editor, edit the exported certificate:
 - Delete ** -----BEGIN CERTIFICATE-----**
 - Delete ** -----END CERTIFICATE-----**
 - Ensure all of the text is a single line

### Step 2: Create an app for ISE in your AAD tenant
1. In the Azure Active Directory (AAD) console, choose **Applications** > **Add an Application** > **Add an application my organization is developing**.
2. Provide a name and URL for the app. The URL could be your company website.
3. Download the app manifest (a JSON file).
4. Edit the manifest JSON file. In the setting called **keyCredentials**, provide the edited certificate text from Step 1 as the setting value.
5. Save the file without changing its name.
6. Provide your App with permissions to the Microsoft Graph and the Microsoft Intune API.
	1. For Microsoft Graph, choose the following
        - **Application permissions**: Read directory data
        - **Delegated permissions**: 
            - Access user’s data anytime
          - Sign users in
   2. For the Microsoft Intune API, in **Application permissions**, choose **Get device state and compliance from Intune**.

7. Choose **View Endpoints** and copy the following values for use in configuring ISE settings:

|Value in AAD portal|Corresponding field in ISE portal|
|-------------------|---------------------------------|
|Microsoft Azure AD Graph API endpoint|Auto Discovery URL|
|Oauth 2.0 Token endpoint|Token Issuing URL|
|Update your code with your Client ID|Client ID|


### Step 3: Configure ISE Settings 
2. In the ISE admin console, provide these setting values: 
  - **Server Type**: Mobile Device Manager
  - **Authentication type**: OAuth – Client Credentials
  - **Auto Discovery**: Yes
  - **Auto Discover URL**: Enter value from Step 1
  - **Client ID**: Enter value from Step 1
  - **Token issuing URL**: Enter value from Step 1



## Information shared between your Intune tenant and your Cisco ISE server
This table lists the information shared between your Intune tenant and your Cisco ISE server for devices that are managed by Intune.

|Property|	Description|
|---------------|------------------------------------------------------------|
|complianceState|	True or false (string) indicating whether device is compliant or non-compliant.|
|isManaged|	True or false (indicating whether the client is managed by Intune or not.|
|macAddress|Mac Address of the device.|
|serialNumber|Serial number of the device. Applies only to iOS Devices.|
|imei|The IMEI (15 decimal digits: 14 digits plus a check digit) or IMEISV (16 digits) includes information on the origin, model, and serial number of the device. The structure of the IMEI/SV is specified in 3GPP TS 23.003. Applies only to devices with SIM cards.)|
|udid|The unique device identifier, a sequence of 40 letters and numbers that is specific to iOS devices.|
|meid|Mobile equipment identifier, a globally-unique number identifying a physical piece of CDMA mobile station equipment. The number format is defined by the 3GPP2 report S. R0048 but in practical terms, it can be seen as an IMEI but with hexadecimal digits. An MEID is 56 bits long (14 hex digits). It consists of three fields, including an 8-bit regional code (RR), a 24-bit manufacturer code, and a 24-bit manufacturer-assigned serial number.| 
|osVersion|	Operating system version for the device.
|model|Device model.
|manufacturer|Device manufacturer.
|azureDeviceId|	The device ID after it has work-place joined with Azure Active Directory. Will be an empty guid for devices that are not joined.|
|lastContactTimeUtc|The date and time when the device last checked in with the Intune management service. 


## User experience

When a user attempts to access resources using an unenrolled device, they will receive a prompt to enroll, such as the one shown here:

![Example of enrollment prompt](../media/cisco-ise-user-iphone.png)

When the user chooses enroll, they are redirected to the Intune enrollment process. The user enrollment experience for Intune is described in these topics:

- [Enroll your Android device in Intune](/intune/end-user/enroll-your-device-in-Intune-android)</br>
- [Enroll your iOS device in Intune](/intune/end-user/enroll-your-device-in-intune-ios)</br>
- [Enroll your Mac OS X device in Intune](/intune/end-user/enroll-your-device-in-intune-mac-os-x)</br>
- [Enroll your Windows device in Intune](/intune/end-user/enroll-your-device-in-intune-windows)</br> 

There is also a [downloadable set of enrollment instructions](https://gallery.technet.microsoft.com/End-user-Intune-enrollment-55dfd64a) that you can use to create customized guidance for your user experience.


### See also

[Cisco Identity Services Engine Administrator Guide, Release 2.1](http://www.cisco.com/c/en/us/td/docs/security/ise/2-1/admin_guide/b_ise_admin_guide_21/b_ise_admin_guide_20_chapter_01000.html#task_820C9C2A1A6647E995CA5AAB01E1CDEF)

