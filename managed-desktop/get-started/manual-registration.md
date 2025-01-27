---
title: Manual registration for new devices
description: Register devices to be managed by Microsoft Managed Desktop
ms.service: m365-md
author: tiaraquan
f1.keywords:
- NOCSH
ms.author: tiaraquan
ms.localizationpriority: medium
ms.collection: M365-modern-desktop
manager: dougeby
ms.topic: article
audience: Admin
msreviewer: andredm7
---

# Manual registration for new devices

Microsoft Managed Desktop can work with brand-new devices, or you can reuse devices you might already have. If you reuse devices, you must reimage them. You're able to register devices with Microsoft Managed Desktop in the Microsoft Endpoint Manager portal.

> [!NOTE]
> Working with a partner to obtain devices? If so, you don't need to worry about getting the hardware hashes; they'll take care of that for you. Make sure your partner establishes a relationship with you at the [Partner Center](https://partner.microsoft.com/dashboard). Your partner can learn more at [Partner Center help](/partner-center/request-a-relationship-with-a-customer). <br><br>Once this relationship established, your partner will simply register devices on your behalf – no further action required from you. If you want to see the details, or your partner has questions, see [Partner registration](partner-registration.md). Once the devices are registered, you can proceed with [checking the image](#check-the-image) and [delivering the devices](#deliver-the-device) to your users.

## Prepare to register brand-new devices

Once you have the new devices in hand, you'll follow these steps:

1. [Obtain the hardware hash for each device.](#obtain-the-hardware-hash)
2. [Merge the hash data](#merge-hash-data).
3. [Register the devices in Microsoft Managed Desktop](#manually-register-devices-in-the-microsoft-managed-desktop-devices-blade).
4. [Double-check that the image is correct.](#check-the-image)
5. [Deliver the device](#deliver-the-device).

### Obtain the hardware hash

Microsoft Managed Desktop identifies each device uniquely by referencing its hardware hash. You have three options for getting this information.

**To obtain the hardware hash:**

- Ask your OEM supplier for the AutoPilot registration file, which will include the hardware hashes.
- Run a [Windows PowerShell script](#powershell-script-method) on each device and collect the results in a file.
- Start each device, but don't complete the Windows setup experience, and [collect the hashes on a removable flash drive](#flash-drive-method).

#### PowerShell script method

You can use the [Get-WindowsAutoPilotInfo.ps1](https://www.powershellgallery.com/packages/Get-WindowsAutoPilotInfo) PowerShell script on the PowerShell Gallery website. For more information about device identification and hardware hash, see [Adding devices to Windows Autopilot](/mem/autopilot/add-devices#device-identification).

**To use the Powershell script method:**

1. Open a PowerShell prompt with administrative rights.
2. Run `Install-Script -Name Get-WindowsAutoPilotInfo`.
3. Run `powershell -ExecutionPolicy Unrestricted`.
4. Run `Get-WindowsAutoPilotInfo -OutputFile <path>\hardwarehash.csv`.
5. Run `powershell -ExecutionPolicy restricted` to prevent subsequent unrestricted scripts from running.

#### Flash drive method

**To use the flash drive method:**

1. On a device other than the one you're registering, insert a USB drive.
2. Open a PowerShell prompt with administrative rights.
3. Run `Save-Script -Name Get-WindowsAutoPilotInfo -Path <pathToUsb>`
4. Turn on the device you're registering, but *don't start the setup experience*. If you accidentally start the setup experience, you'll have to reset or reimage the device.
5. Insert the USB drive, and then press SHIFT + F10.
6. Open a PowerShell prompt with administrative rights, and then run `cd <pathToUsb>`.
7. Run `Set-ExecutionPolicy -ExecutionPolicy Unrestricted`
8. Run `.\Get-WindowsAutoPilotInfo -OutputFile <path>\hardwarehash.csv`
9. Remove the USB drive, and then shut down the device by running `shutdown -s -t 0`

> [!IMPORTANT]
> Do not power on the device you are registering again until you've completed registration for it.

### Merge hash data

You'll need to have the data in the CSV files combined into a single file to complete registration. Here's a sample PowerShell script to make it easy:

`Import-CSV -Path (Get-ChildItem -Filter *.csv) | ConvertTo-Csv -NoTypeInformation | % {$_.Replace('"', '')} | Out-File .\aggregatedDevices.csv`

> [!NOTE]
> Extra columns are not supported. Quotes are not supported. Only ANSI-format text files can be used (not Unicode). Headers are case-sensitive. Editing the file in Excel and saving it as a CSV file will not generate a usable file due to these requirements. Be sure to preserve any leading zeroes in the device serial numbers.

### Manually register devices in the Microsoft Managed Desktop Devices blade

**To manually register devices in the Microsoft Managed Desktop Devices blade:**

1. Collect the [hardware hash](../get-started/manual-registration.md#obtain-the-hardware-hash) for new devices.
2. Go to the [Microsoft Endpoint Manager portal](https://endpoint.microsoft.com) and select **Devices** in the left navigation pane.
3. In the **Microsoft Managed Desktop** section, select **Devices**.
4. In the **Microsoft Managed Desktop Devices** workspace, select **+ Register devices**. A fly-in menu opens to register new devices.
5. In the **File upload** box, provide the path to the CSV file you created previously.
6. Select a **[device profile](../service-description/profiles.md)** from the dropdown menu. You can only select one device profile at a time (Standard, SensitiveData or PowerUser).
7. Optional. Turn on the **[Shared device](../service-description/shared-devices.md)** toggle if you want to register Shared devices with Microsoft Managed Desktop.
7. Select **Register devices**. The system will add the devices to your list of devices. If successful, the device will show as **Ready for user** meaning it's ready and waiting for a user to start using.
8. If registering Shared devices, the **Shared Device** column will be marked with **Yes**.

> [!NOTE]
> If you manually change the Azure Active Directory (AAD) group membership of a device, it will be automatically reassigned to the group for its device profile and removed from any conflicting groups.

You can monitor the progress of device registration on the main page. Possible states reported include:

| State | Description |
| -----|-----|
| Registration Pending | Registration isn't completed yet. Check back later. |
| Registration failed | Registration couldn't be completed. For more information, see [Troubleshooting device registration](#troubleshooting-device-registration). |
| Ready for user | Registration succeeded. The device is now ready to be delivered to the user. Microsoft Managed Desktop will guide them through first-time set-up, so there's no need for you to do any further preparations. |
| Active | The device has been delivered to the user and they've registered with your tenant. This state also indicates that they're regularly using the device. |
| Inactive | The device has been delivered to the user and they've registered with your tenant. However, they haven't used the device recently (in the last seven days).  |

#### Troubleshooting device registration

| Error message | Details |
|-----| ----- |
| Device not found | We couldn't register this device because we couldn't find a match for the provided manufacturer, model, or serial number. Confirm these values with your device supplier. |
| Hardware hash not valid | The hardware hash you provided for this device wasn't formatted correctly. Double-check the hardware hash and then resubmit. |
| Device already registered | This device is already registered to your organization. No further action required. |
| Device claimed by another organization | This device has already been claimed by another organization. Check with your device supplier. |
| Unexpected error | Your request couldn't be automatically processed. Contact Support and provide the Request ID: `<requestId>` |

### Check the image

If your device has come from a Microsoft Managed Desktop partner supplier, the image should be correct.

You're also welcome to apply the image on your own if you prefer. To get started, contact the Microsoft representative you're working with. The representative will provide you the location and steps for applying the image.

### Autopilot group tag

When you use the Admin portal to register devices, we automatically assign the Autopilot Group Tag associated with the device profile listed in [Register devices by using Partner Center](partner-registration.md).
The service monitors all Microsoft Managed Desktop devices daily and assigns the group tag to any that don't already have it.

### Deliver the device

> [!IMPORTANT]
> Before you hand off the device to your user, make sure you have obtained and applied the [appropriate licenses](../get-ready/prerequisites.md) for that user.

If all the licenses are applied, you can [get your users ready to use devices](get-started-devices.md). Then, your user can start up the device and proceed through the Windows setup experience.
