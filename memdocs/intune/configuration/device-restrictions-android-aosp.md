---
# required metadata

title: Device restriction settings for Android (AOSP) platform in Microsoft Intune
description: Use Android (AOSP) restriction settings to control a wide range of settings and features on AOSP devices.  
keywords:
author: MandiOhlinger
ms.author: mandia
manager: dougeby
ms.date: 10/19/2021
ms.topic: conceptual
ms.service: microsoft-intune
ms.subservice: configuration
ms.localizationpriority: medium
ms.technology:
# optional metadata

#ROBOTS:
#audience:

params:
  siblings_only: true
ms.reviewer: mikedano, chmaguir, chrisbal, priyar
ms.suite: ems
search.appverid: MET150
#ms.tgt_pltfrm:
ms.custom: intune-azure, seodec18
ms.collection: M365-identity-device-management
---

# Android (AOSP) device settings to allow or restrict features using Intune

*This feature is in public preview.*

This article describes the different settings you can control on Android (AOSP) devices. You can use these restrictions to configure password requirements and access to device features. 

This feature applies to the following Android (AOSP) device types: 

- Corporate owned userless devices (shared) 
- Corporate-owned user-associated devices (single user)  

## Before you begin

Create an [Android (AOSP) device restrictions profile](device-restrictions-configure.md). For **Platform**, select **Android (AOSP)**.    

## Device password  

- **Required password type**: Choose if a password should include only numeric characters, or a mix of numerals and other characters. Your options
  - **Device default**: To evaluate password compliance, be sure to select a password strength other than **Device default**. If you want to require users to set up a passcode on their devices, configure this setting to a more secure option.
  - **Numeric** (default): Password must only be numbers, such as `123456789`. Also enter:
    - **Minimum password length**: Enter the minimum length the password must have, between 4 and 16 characters.
  - **Numeric complex**: Repeated or consecutive numbers, such as "1111" or "1234", aren't allowed. Also enter:
    - **Minimum password length**: Enter the minimum length the password must have, between 4 and 16 characters.
- **Minimum password length**: Enter the minimum number of characters required, from 4-16. For example, enter `6` to require at least six numbers or characters in the password length. 

> [!NOTE]  
>- RealWear devices currently only support device default, numeric, and numeric complex password types.  
>- The password type **Password required, no restrictions** appears as an option but doesn't currently work on devices, which is a known issue.  


## General 

- **Block Access to Camera**: Prevents access to the camera on the device. When set to **Not configured** (default), Intune doesn't change or update this setting. By default, the OS might allow access to the camera.  

  Intune only manages access to the device camera. It doesn't have access to pictures or videos.  

- **Block Screen Capture**: Prevents screenshots or screen captures on the device. It also prevents the content from being shown on display devices that don't have a secure video output. When set to **Not configured** (default), Intune doesn't change or update this setting. By default, the OS might let users capture the screen contents as an image.  

- **Disable Factory Reset**: Prevents users from using the factory reset option in the device's settings.  When set to **Not configured** (default), Intune doesn't change or update this setting. By default, the OS might allow external media on the device. 

- **Block mounting of external media**: Prevents users from using or connecting any external media on the device.  When set to **Not configured** (default), Intune doesn't change or update this setting. By default, the OS might allow users to connect external media.  

- **Block USB file transfer**: Prevents users from transferring files over USB. When set to **Not configured** (default), Intune doesn't change or update this setting. By default, the OS might allow users to transfer files.  

- **Block Wi-Fi setting changes**: Prevents users from creating or changing any Wi-Fi configurations. When set to **Not configured** (default), Intune doesn't change or update this setting. By default, the OS might allow users to change the Wi-Fi settings on the device.  

- **Disable Bluetooth**: Disables Bluetooth on the device so that users can't pair with other devices. When set to **Not configured** (default), Intune doesn't change or update this setting. By default, the OS might enable Bluetooth on the device.   

- **Block Bluetooth configuration**: Prevents users from configuring Bluetooth on the device. When set to **Not configured** (default), Intune doesn't change or update this setting. By default, the OS might allow users to configure Bluetooth.   

- **Allow users to turn on debugging features**: Permits users to access the debugging features on the device. When set to **Not configured** (default), Intune doesn't change or update this setting. By default, the OS might prevent users from using the debugging features on the device.  

## Next steps  

- [Create an Android (AOSP) device compliance policy](../protect/compliance-policy-create-android-aosp.md).   

- [Add actions for noncompliant devices](../protect/actions-for-noncompliance.md).  

