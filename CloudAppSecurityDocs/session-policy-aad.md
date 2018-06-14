---
# required metadata

title: Create session policies to gain deep visibility into user session activities and block downloads | Microsoft Docs
description: This topic describes the procedure for setting up a Cloud App Security Conditional Access App Control session policy gain deep visibility into user session activities and block downloads using reverse proxy capabilities.
keywords:
author: rkarlin
ms.author: rkarlin
manager: mbaldwin
ms.date: 5/14/2018
ms.topic: article
ms.prod:
ms.service: cloud-app-security
ms.technology:
ms.assetid: 745df28a-654c-4abf-9c90-203841169f90

# optional metadata

#ROBOTS:
#audience:
#ms.devlang:
ms.reviewer: reutam
ms.suite: ems
#ms.tgt_pltfrm:
#ms.custom:

---
*Applies to: Microsoft Cloud App Security*

# Session policies 

> [!NOTE]
> This is a preview feature.


>[!div class="step-by-step"]
[« PREVIOUS: Deploy Conditional Access App Control](proxy-deployment-aad.md)<br>
[NEXT: How to create an access policy »](access-policy-aad.md)


Microsoft Cloud App Security session policies enable real-time session-level monitoring, affording you granular visibility into cloud apps and the ability to take different actions depending on the policy you set for a user session. Instead of [allowing or blocking access completely](access-policy-aad.md), with session control you can allow access while monitoring the session and/or limit specific session activities using the reverse proxy capabilities of Conditional Access App Control. 

For example, you can decide that from unmanaged devices, or for sessions coming from specific locations, you want to allow the user to access the app, but also limit the download of sensitive files or require that certain documents be protected upon download. Session policies enable you to set these user-session controls and allow access and enables you to:

- [Monitor all activities](#monitor-session)
- [Block all downloads](#block-download)
- [Block specific activities](#block-activities)
- [Protect files on download](#protect-download)
 

## Prerequisites to using session policies

- Azure AD Premium P2 license
- The relevant apps should be [deployed with Conditional Access App Control](proxy-deployment-aad.md)
- An [Azure AD conditional access policy](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-azure-portal) should be in place that redirects users to Microsoft Cloud App Security, as described below.

> [!NOTE]
> - Session policies also support apps that are configured with identity providers other than Azure AD in Private Preview. For more information about the Private Preview, send an email to mcaspreview@microsoft.com.

## Create an Azure AD conditional access policy

Azure Active Directory conditional access policies and Cloud App Security session policies work in tandem to examine each user session and make policy decisions for each app. To set up a conditional access policy in Azure AD, follow this procedure:

1. Configure an [Azure AD conditional access policy](https://docs.microsoft.com/azure/active-directory/active-directory-conditional-access-azure-portal) with assignments for user or group of users and the SAML app you want to control with the Conditional Access App Control. 

   > [!NOTE]
   > Only apps that were [deployed with Conditional Access App Control](proxy-deployment-aad.md) will be affected by this policy.

2. Route users to Microsoft Cloud App Security by selecting the **Use Conditional Access App Control enforced restrictions** in the **Session** blade.

   ![Conditional Access App Control restrictions Azure AD conditional access](./media/proxy-deploy-restrictions-aad.png)

## Create a Cloud App Security session policy 

To create a new session policy, follow this procedure:

1. In the portal, select **Control** followed by **Policies**.
2. In the **Policies** page, click **Create policy** and select **Session policy**.  

   ![Create session policy](./media/create-session-policy.png)

3. In the **Session policy** window, assign a name for your policy, such as *Block Download of Sensitive Documents in Box for Marketing Users*.

   ![New session policy](./media/new-session-policy.png)

4. In the **Session control type** field: 

   1. Select **Monitor only** if you only want to monitor activities by users. This will create a Monitor only policy in which, all sign-ins, heuristic downloads, and Activity types will be downloaded, for the apps you selected.

   2. Select **Control file download (with DLP)** if you want to monitor user activities and take additional actions, such as block or protect downloads for users.

      ![session policy control type](./media/session-policy-control-type.png)
   
   3. Select **Block activities** to block specific activities which you can select using the **Activity type** filter. All activities from selected apps  will be monitored (and reported in the Activity log). The specific activities you select will be blocked if you select the **Block** action, and the specific activities you selected will raise alerts on if you select the **Test** action and have alerts turned on.

1. Under **Activity source** in the **Activities matching all of the following** section, select additional activity filters to apply to the policy. These can include the following options: 

   - **Device tags**: Use this filter to identify unmanaged devices.

   - **Location**: Use this filter to identify unknown (and therefore risky) locations. 

   - **IP address**: Use this filter to filter per IP addresses or use previously assigned IP address tags. 

   - **User agent tag**: Use this filter to enable the heuristic to identify mobile and desktop apps. This filter can be set to equals or does not equal **Native client** and should be tested against your mobile and desktop apps for each cloud app.
         
       ![native client support](./media/user-agent-tag.png)

     >[!NOTE]
     >Session policies don’t support mobile and desktop apps. Make sure to test session policies to see that they don’t interfere with mobile and desktop app functionality. If necessary, exclude mobile and desktop apps from session policies.

     ![session policy activity source](./media/session-policy-activity-filters.png)

6. If you selected the option to **Control file download (with DLP)**:

   1. Under **Activity source** in the **Files matching all of the following** section, select additional file filters to apply to the policy. These can include the following options:

      - **Classification label** - Use this filter if your organization uses Azure Information Protection, and your data has been protected by its Classification labels. Here you will then be able to filter files based on the Classification label you applied to them. For more information about integration between Cloud App Security and Azure Information Protection, see [Azure Information Protection integration](azip-integration.md).

      - **File name** - Use this filter to apply the policy to specific files.

      - **File type** - Use this filter to apply the policy to specific file types, for example, block download for all .xls files.

        ![session policy file filters](./media/session-policy-file-filters.png)

        
   2. In the **Content inspection** section, set whether you want to enable the DLP engine to scan documents and file content.
 
      ![session policy content inspection](./media/session-policy-content-inspection.png)

   3. Under **Actions**, select one of the following: 

      - **Test (Monitor all activities)**: Set this action to explicitly allow download according to the policy filters you set.

      - **Block (Block file download and monitor all activities)**: Set this action to explicitly block download according to the policy filters you set. For more information, see [How block download works](#block-download).

      - **Protect (Apply classification label to download and monitor all activities)**: This is only available if you selected **Control file download (with DLP)** under **Session policy**. If your organization uses Azure Information Protection, you can set an **Action** to apply a classification label set in Azure Information Protection to the file. For more information, see [How protect download works](#protect-download).

        ![session policy actions](./media/session-policy-actions.png)

7. You can **Create an alert for each matching event with the policy's severity** and set an alert limit and select whether you want the alert as an email, a text message or both.

   ![session policy alert](./media/session-policy-alert.png)


## Monitor all activities <a name="monitor-session"></a>

When you create a session policy, each user session that matches the policy is redirected to session control rather than to the app directly. The user will see a monitoring notice to let them know that their sessions are being monitored.

   ![session monitoring notice](./media/session-monitoring-notice.png)

If you do not want to notify the user that they are being monitored, you can disable the notification message.

1. Under the settings cog, select **General settings**. 

2. Then, under **Conditional Access App Control** select **User monitoring** and unselect the **Notify users** checkbox.

    ![disable session monitoring notice](./media/disable-session-monitoring-notice.png)

To keep the user within the session, Conditional Access App Control replaces all the relevant URLs, Java scripts, and cookies within the app session with Microsoft Cloud App Security URLs. For example: if the app returns a page with links whose domains ends with myapp.com, Conditional Access App Control replaces the links with domains ending with something like: myapp.com.us.cas.ms. This way the entire session is monitored by Microsoft Cloud App Security.

Conditional Access App Control records the traffic logs of every user session that is routed through it. The traffic logs include the time, IP, user agent, URLs visited, and the number of bytes uploaded and downloaded. These logs are analyzed and a continuous report called **Cloud App Security Conditional Access App Control** is added to the list of Cloud Discovery reports in the Cloud Discovery dashboard.

![Conditional Access App Control report](./media/proxy-report.png)


To export these logs:

1. Go to the settings cog and click **Conditional Access App Control**.
2. On the right side of the table, click the export button ![export button](./media/export-button.png). 
3. Select the range of the report and click **Export**. This process may take some time.

To download the exported log:

1. After the report is ready, go to **Investigate** and then **Custom reports**.
2. In the table, select the relevant report from the list of **Conditional Access App Control traffic logs** and click download ![download button](./media/download-button.png). 


## Block all downloads <a name="block-download"></a>

When **Block** is set as the **Action** you want to take in the Cloud App Security session policy, Conditional Access App Control prevents a user from downloading a file in accordance with the policy’s file filters. A download event is recognized by Microsoft Cloud App Security for each SAML app and when a user initiates this event, Conditional Access App Control intervenes in real time to prevent it from running. When the signal is received that a user has initiated a download, Conditional Access App Control returns a **Download restricted** message to the user and replaces the downloaded file with a text file that contains a customizable message to the user, which can be configured from the session policy.  

## Block specific activities <a name="block-activities"></a>

When **Block activities** is set as the **Activity type**, you can select specific activities to block in specific apps. All activities from selected apps will be monitored (and reported in the Activity log) and the specific activities you select will be blocked if you select the **Block** action, and the specific activities you selected will raise alerts on if you select the **Test** action and have alerts turned on.

## Protect files on download <a name="protect-download"></a>
Select **Block activities** to block specific activities which you can select using the **Activity type** filter. All activities from selected apps  will be monitored (and reported in the Activity log). The specific activities you select will be blocked if you select the **Block** action, and the specific activities you selected will raise alerts on if you select the **Test** action and have alerts turned on.
When **Protect** is set as the **Action** to be taken in the Cloud App Security session policy, Conditional Access App Control enforces the labeling and subsequent protection of a file in accordance with the policy’s file filters. Labels are configured in the Azure Information Protection console in Azure and **Protect** must be selected within the label for the label to appear as an option in the Cloud App Security policy. When a label is selected, and a file is downloaded that meets the criteria of the Cloud App Security policy, the label, and corresponding protection (with permissions) is applied to the file upon download. The original file remains as-is in the cloud app while the downloaded file is now protected. Users who attempt to access the file must meet the permission requirements determined by the protection applied.  
 
>[!div class="step-by-step"]
[« PREVIOUS: Deploy Conditional Access App Control](proxy-deployment-aad.md)<br>
[NEXT: How to create an access policy »](access-policy-aad.md)

 
## See Also  
[Blocking downloads on unmanaged devices using Azure AD Conditional Access App Control capabilities](use-case-proxy-block-session-aad.md)   

[Premier customers can also choose Cloud App Security directly from the Premier Portal.](https://premier.microsoft.com/)  
  
  