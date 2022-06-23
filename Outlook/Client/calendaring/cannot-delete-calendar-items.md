---
title: Unable to delete calendar items
description: You can't delete corrupted calendar items in Outlook. Even if you use the MFCMAPI or EWSEditor tool, the corrupted items are still not deleted.
author: MaryQiu1987
ms.author: v-maqiu
manager: dcscontentpm
audience: ITPro
ms.topic: troubleshooting
localization_priority: Normal
ms.custom: 
  - Outlook for Windows
  - CI 161181
  - CSSTroubleshoot
ms.reviewer: nourdinb, tylewis
appliesto: 
  - Exchange Server 2013
  - Exchange Server 2016
  - Exchange Server 2019
  - Exchange Online
search.appverid: MET150
ms.date: 6/14/2022
---
# Can't delete calendar items in Outlook

## Symptoms

When you try to delete a calendar item by using Microsoft Outlook in online mode, you receive the following error message:

> The move, copy, or deletion cannot be completed. The items might have been moved or deleted, or you may not have sufficient permission. If the item was sent as a task request or meeting request, the sender might not receive updates.  

If you try to delete the item by using Outlook in Cached Exchange mode, it appears as deleted for a brief time before appearing again.

Also, you're unable to delete the item by using the MFCMAPI and EWSEditor tools. See the [Details](#details) section for more information.

## Cause

This issue occurs because the calendar item is corrupted. When a calendar item in a mailbox is deleted, the change is logged in the Calendar Logging folder. If the item is corrupted, then the logging is triggered but doesn't execute correctly and an exception is raised. This prevents the deletion from succeeding.

## Resolution

To resolve this issue, temporarily prevent the change to the calendar item from being logged and then delete the item.

1. Run the following cmdlet:

    ```powershell
    Set-Mailbox <name_of_affected_mailbox> -CalendarVersionStoreDisabled $true 
    ```

    Then wait for the database store configuration cache to expire. This will take about two hours. Then proceed to step 3.

1. As an alternate to waiting for the cache to expire, if the affected mailbox is on Exchange on-premises, you can use one of the following options and then proceed to step 3. However, they will cause service interruptions.

    - Restart the Exchange Information Store service.
    - Mount the affected user's database on another Exchange server.

1. Delete the calendar item. We recommend that you use the [MFCMAPI](https://github.com/stephenegriffin/mfcmapi/releases/) tool.

1. After the item is deleted, run the following cmdlet to reverse the change to the value of the `CalendarVersionStoreDisabled` parameter:

    ```powershell
    Set-Mailbox <name_of_affected_mailbox> -CalendarVersionStoreDisabled $false
    ```

## Details

### Delete the calendar item by using the MFCMAPI tool

When you open the calendar item to delete in [MFCMAPI](https://github.com/stephenegriffin/mfcmapi/releases/), you see a limited number of MAPI properties only. This indicates that the item is corrupted.

In the following screenshot, only 21 properties are displayed for a corrupted calendar item:

:::image type="content" source="media/cannot-delete-calendar-items/mfcmapi.png" alt-text="Screenshot of a Calendar item example showing in MFCMAPI, which has 21 MAPI properties." border="false":::

When you right-click the item, select **Delete message**, and then from the drop-down menu for **Deletion style**, you select **Permanent deletion (deletes to deleted item retention if supported)** and select **OK**, you receive the following warning message:

> Warning:  
> Code: MAPI_W_PARTIAL_COMPLETION == 0x00040680  
> Function m_IpFolder->DeleteMessages(IpEIDs, IpProgress ? reinterpret_cast\<ULONG_PTR>(m_hWnd) : NULL, IpProgress, uIFlag)  
> File D:\a\1\s\UI\Dialogs\ContentsTable\FolderDlg.cpp  
> Line 678

Alternatively, when you select **Permanent delete passing DELETE_HARD_DELETE (unrecoverable)** from the drop-down menu for **Deletion style** and select **OK**, the tool doesn't respond and the item isn't deleted.

### Delete the calendar item by using the EWSEditor tool

When you open the calendar item you want to delete by using the [EWSEditor](https://github.com/dseph/EwsEditor/releases) tool, you receive the following error message:

> ErrorCode: ErrorContentConversionFailed  
> ErrorMessage: Content conversion failed. Content conversion: Body conversion failed.  

If you select **OK** in the error message, the calendar item is displayed in the tool but either a limited number of properties or no properties are displayed for the item as shown in the following screenshot:

:::image type="content" source="media/cannot-delete-calendar-items/ewseditor.png" alt-text="Screenshot of a Calendar item example showing in EWSEditor, which has no properties.":::

When you right-click the item to delete it, you see an exception message. The following text is a snippet of the message:

> Exception details:  
> Message: Content conversion failed. Content conversion: Body conversion failed.  
> Type: Microsoft.Exchange.WebServices.Data.ServiceResponseException  
> Source: Microsoft.Exchange.WebServices  
> ErrorCode: ErrorContentConversionFailed  
> ErrorMessage: Content conversion failed. Content conversion: Body conversion failed.

When you see this exception message, it indicates that the item is corrupted.