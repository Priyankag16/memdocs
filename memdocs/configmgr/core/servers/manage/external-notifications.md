---
title: External notifications
titleSuffix: Configuration Manager
description: Enable the site to send notifications to an external system or application
ms.date: 08/02/2021
ms.prod: configuration-manager
ms.technology: configmgr-core
ms.topic: how-to
author: aczechowski
ms.author: aaroncz
manager: dougeby
ms.localizationpriority: medium
---

# External notifications

*Applies to: Configuration Manager (current branch)*

<!--9504414-->

In a complex IT environment, you may have an automation system like [Azure Logic Apps](/azure/logic-apps/logic-apps-overview). Customers use these systems to define and control automated workflows to integrate multiple systems. You could integrate Configuration Manager into a separate automation system through the product's SDK APIs. But this process can be complex and challenging for IT professionals without a software development background.

Starting in version 2107, you can enable the site to send notifications to an external system or application. This feature simplifies the process by using a web service-based method. You configure [subscriptions](configure-alerts.md) to send these notifications. These notifications are in response to specific, defined events as they occur. For example, [status message filter rules](use-status-system.md#manage-status-filter-rules).

> [!NOTE]
> The external system or application defines and provides the methods that this feature calls.

When you set up this feature, the site opens a communication channel with the external system. That system can then start a complex workflow or action that doesn't exist in Configuration Manager.

These notifications use the following standardized schema:

```json
{
    "properties": {
        "EventID": {
            "type": "integer"
        },
        "EventName": {
            "type": "string"
        },
        "EventPayload": {
            "type": "string"
        },
        "MessageID": {
            "type": "string"
        },
        "ServerName": {
            "type": "string"
        },
        "SiteCode": {
            "type": "string"
        },
        "Source": {
            "type": "string"
        }
    },
    "type": "object"
}
```

## Prerequisites

- The site's service connection point needs to be in online mode. For more information, see [About the service connection point](../deploy/configure/about-the-service-connection-point.md).

- Currently, this feature only supports Azure Logic Apps as the external system. An active Azure subscription with rights to create a logic app is required.

- The **Full administrator** security role with **All** security scope in Configuration Manager.

- In this release, to create the objects in Configuration Manager, you need to use the PowerShell script **SetupExternalServiceNotifications.ps1**. Use the following script sample to properly get the PowerShell script to use for this feature:<!-- 10363343 -->

    ```powershell
    $FileName = ".\SetupExternalServiceNotifications.ps1"
    Invoke-WebRequest https://aka.ms/cmextnotificationscript -OutFile $FileName
    (Get-Content $FileName -Raw).Replace("`n","`r`n") | Set-Content $FileName -Force
    (Get-Content $FileName -Raw).TrimEnd("`r`n") | Set-Content $FileName -Force
    ```

    > [!NOTE]
    > **SetupExternalServiceNotifications.ps1** is digitally signed by Microsoft. This script sample downloads the file and fixes the line breaks to preserve the digital signature.

## Create an Azure logic app and workflow

Use the following process to create a sample app in Azure Logic Apps to receive the notification from Configuration Manager.

> [!NOTE]
> This process is provided as an example to help you get started. It's not intended for production use.

1. Sign in to the [Azure portal](https://portal.azure.com).

1. In the Azure search box, enter `logic apps`, and select **Logic Apps**.

1. Select **Add** and choose **Consumption**. This action creates a new logic app.

1. On the **Basics** tab, specify the project details as necessary for your environment: subscription name, resource group, logic app name, and region.

1. Select **Review + create**. On the validation page, confirm the details that you provided, and select **Create**.

1. Under **Next steps**, select **Go to resource**.

1. Under the section to **Start with a common trigger**, select **When a HTTP request is received**.

1. At the bottom of the trigger editor, select **Use sample payload to generate schema**.

1. Paste the following sample schema:

    ```json
    {
        "EventID":0,
        "EventName":"",
        "SiteCode":"",
        "ServerName":"",
        "MessageID":0,
        "Source":"",
        "EventPayload":""
    }
    ```

1. Select **Done** and then select **Save**.

1. Copy the generated URL for the logic app. You'll use this URL later when you create the subscription in Configuration Manager.

1. To add a new step in the designer, select **+ New Step**. Choose an appropriate action when it receives a notification from Configuration Manager. For example:

    - To send an email, use the [Office 365 Outlook connector](/azure/connectors/connectors-create-api-office365-outlook).
    - To post a message to Teams, use the [Microsoft Teams connector](/connectors/teams/).

    Sign in if necessary and complete the required information for the action. For more information, see the [Create logic apps](/azure/logic-apps/quickstart-create-first-logic-app-workflow#add-an-action) quickstart in the Azure Logic Apps documentation.

## Create an event in Configuration Manager

There are two types of events that are currently supported:

- The site raises a status message that matches conditions specified in a status filter rule.

- A user requests approval for an application in Software Center.

### Status message

1. On the site server, run **SetupExternalServiceNotifications.ps1**. Since you're running it on the site server, enter `y` to continue.

1. Select option `2` to create a new status filter rule.

1. Specify a name for the new status filter rule.

1. Select message-matching criteria for the rule, and specify values to match. Specify `0` to not use a criterion.

    The following criteria are available:

    - **Source**: Client, SMS Provider, Site Server
    - **Site code**
    - **System**
    - **Component**
    - **Message type**: Milestone, Detail, Audit
    - **Severity**: Informational, Warning, Error
    - **Message ID**
    - **Property**
    - **Property value**

    For more information about criteria for status message rules, see [Use the status system](use-status-system.md).

    > [!IMPORTANT]
    > Be cautious with the type of status filter rule that you create. For external notifications, the site can process 300 status messages every five minutes. If your rule allows more messages than this limit, it will cause a backlog on the site. Create rules with narrow filters for specific scenarios. Avoid generic rules that allow a lot of messages.

1. Rerun the PowerShell script. Select option `3` to create a new subscription.

1. Specify a name and description for the subscription. Then specify the logic app URL that you previously copied from the Azure portal.

1. Select the new status filter rule.

1. Select `0` to exit the script.

1. Trigger an event for the site component you chose for the status filter rule.

    For example, use the [Configuration Manager Service Manager](../deploy/configure/site-components.md#configuration-manager-service-manager) to restart the component.

### App approval

> [!NOTE]
> This event type requires a application that requires approval and is deployed to a user collection. For more information, see [Deploy applications](../../../apps/deploy-use/deploy-applications.md) and [Approve applications](../../../apps/deploy-use/app-approval.md).

1. On the site server, run **SetupExternalServiceNotifications.ps1**. Since you're running it on the site server, enter `y` to continue.

1. Select option `3` to create a new subscription.

1. Specify a name and description for the subscription. Then specify the logic app URL that you previously copied from the Azure portal.

1. Select the appropriate event for an application request.

1. Select `0` to exit the script.

1. On a managed device, submit an app approval request from Software Center. For more information, see [Software Center user guide](../../understand/software-center.md#install-an-application).

## Monitor the workflow

Within five minutes, the event triggers the logic app workflow. Check the status of the workflow in the Azure portal. Navigate to the **Runs history** page of the logic app.

For more information, see [Monitor run status, review trigger history, and set up alerts for Azure Logic Apps](/azure/logic-apps/monitor-logic-apps).

## Troubleshoot

Use the following Configuration Manager log files on the site server to help troubleshoot this process:

- **ExternalNotificationsWorker.log**: Check if the queue has been processed and notifications are sent to external system.
- **statmgr.log**: Check if the status filter rules have been processed without errors

## Remove a subscription

If you need to delete a subscription, use the following process:

1. Run the **SetupExternalServiceNotifications.ps1** script with option `1` to list the available subscriptions. Note the subscription ID, which is an integer value.

1. Use the **NotificationSubscription** API of the administration service. Make a DELETE call to the URI `https://<SMSProviderFQDN>/AdminService/v1.0/NotificationSubscription/<Subscription_ID>`.

    For more information, see [How to use the administration service in Configuration Manager](../../../develop/adminservice/usage.md).

After you remove the subscription, the site doesn't send notifications to the external system.

## Known issues

If you create a [status filter rule](#status-message), you'll see it in the site's list of **Status filter rules** in the Configuration Manager console. If you make a change on the **Actions** tab of the rule properties, the external notification won't work.

In version 2107, after you [recover a central administration site](recover-sites.md) (CAS), delete and recreate the subscription.<!-- 10333966 -->

> [!TIP]
> Before you [remove a CAS](../deploy/install/remove-central-administration-site.md), recreate the subscriptions at the child primary site.

## Script usage

When you run **SetupExternalServiceNotifications.ps1**, it detects whether it's running on a site server:

- `Y`: Continue on the current server
- `N`: Specify the FQDN of a site server to use

If the script doesn't detect a site server, it prompts for an FQDN.

The following actions are then available:

- `0`: Skip/continue
- `1`: List available subscriptions
- `2`: Create a status filter rule to expose status messages
- `3`: Create a subscription. This option is only available for the top-level site.

> [!NOTE]
> This script is only supported for sites running version 2107 or later.

## Next steps

[Use the status system](use-status-system.md)

[Configure alerts](configure-alerts.md)
