---
title: Develop an Integration app that uses Partial Transcript notifications
author: jacob.shaw
indextype: blueprint
icon: blueprint
image: images/flowchart.png
category: 6
summary: This Genesys Cloud Developer Blueprint demonstrates an example of how Partial Transcript notifications can be used in the event of a Genesys Cloud Integration.
---
:::{"alert":"primary","title":"About Genesys Cloud Blueprints","autoCollapse":false} 
Genesys Cloud blueprints were built to help you jump-start building an application or integrating with a third-party partner. 
Blueprints are meant to outline how to build and deploy your solutions, not a production-ready turn-key solution.
 
For more details on Genesys Cloud blueprint support and practices 
please see our Genesys Cloud blueprint [FAQ](https://developer.genesys.cloud/blueprints/faq)sheet.
:::

![Partial Transcript App flowchart](images/flowchart.png "Partial Transcript App flowchart")

## Solution
`
The sample app included in this blueprint solution is an admin dashboard that allows administrators to view active conversations in their organization's queues. The administrator can look at info about each conversation, including the ongoing transcript, and assign a "standing" to each call. The "standing" is a simple good-or-bad state based on whether the agent said one of the blacklisted phrases. These phrases are defined within `src/components/queue-list/QueueList.tsx` and can be replaced with the phrases of your choosing.

```javascript
const blacklistedPhrases: string[] = ['blacklisted phrase']; // Replace this with the phrases you want to blacklist
```

:::warning: **Warning**: If your organization contains a large number of queues, you may encounter rate limits when subscribing to the conversations for each queue. In this case, one fix is to design your solution as a standalone app rather than an embedded client app. This allows you to initialize and run the app persistently in the background, which prevents re-initializing the embedded client app each time it is opened. Alternatively, you may implement a user-defined filter on the queue list with a maximum list size that prevents the rate limits. This will lengthen the initialization time for the application and could cause unexpected behavior.
:::

The blueprint describes the required steps to develop and integrate this sample app into Genesys Cloud.

## Contents

* [Solution components](#solution-components "Goes to the Solutions components section")
* [Requirements](#requirements "Goes to the Requirements section")
* [Running locally](#running-locally "Goes to the Running locally section")
* [Sample app overview](#sample-app-overview "Overview of the sample app's features")
* [Configuring the React Project to use Genesys Cloud SDK](https://developer.genesys.cloud/blueprints/react-app-with-genesys-cloud-sdk/#create-a-react-project "How to integrate the Genesys Cloud SDK")
* [Additional resources](#additional-resources "Goes to the Additional resources section")

## Solution components

- **Genesys Cloud** - A suite of Genesys cloud services for enterprise-grade communications, collaboration, and contact center management. You deploy the Chat Translator solution in Genesys Cloud.
- **Genesys AppFoundry** - The Genesys app marketplace for solutions that run on the Genesys Cloud platform. You download the integration used in this solution from the Genesys AppFoundry. 
- **About Client Application integration** - The Genesys Cloud integration that embeds third-party webapps via iframe in the Genesys Cloud UI. For more information, see: [About Client Application integrations](https://help.mypurecloud.com/?p=131847 "Goes to About Client Application integrations article") in the Genesys Cloud Resource Center.

## Prerequisites

### Specialized knowledge

* Administrator-level knowledge of Genesys Cloud
* Experience with Typescript or JavaScript
* Experience using the Genesys Cloud Platform API

### Software development kit (SDK)

- **Platform API JavaScript Client** - The sample app employs React+TypeScript; thus, the javaScript SDK is used. However, the same functionality could be achieved using other languages. For more information see, [Platform API Javascript Client](https://developer.genesys.cloud/api/rest/client-libraries/javascript/ "Goes to Platform API Javascript Client page") in the Genesys Cloud Developer Center.
- **Genesys AppFoundry** - The Genesys app marketplace for solutions that run on the Genesys Cloud platform. You download the integration used in this solution from the Genesys AppFoundry. 

### Genesys Cloud account requirements

This solution requires a Genesys Cloud license. For more information see, [Genesys Cloud pricing](https://www.genesys.com/pricing "Goes to Pick the Perfect Plan for your Business page").

A recommended Genesys Cloud role for the solutions engineer is the Master Admin. For more information see, [Roles and permissions overview](https://help.mypurecloud.com/?p=24360 "Goes to Roles and permissions overview article") in the Genesys Cloud Developer Center.

## Running locally

### Download the repository that contains the project files

For more information see, [Partial Transcription Blueprint](https://github.com/GenesysCloudBlueprints/partial-transcription-blueprint "Goes to Partial Transcription Blueprint page") in the GitHub repository.

```bash
git clone https://github.com/GenesysCloudBlueprints/partial-transcription-blueprint.git
```

For more information, see: [Partial Transcription Blueprint](https://github.com/GenesysCloudBlueprints/partial-transcription-blueprint "Goes to Partial Transcription Blueprint page") in the GitHub repository.

### Create an Implicit Grant OAuth

1. Log in to your Genesys Cloud organization and create a new OAuth Credential (Implicit Grant). [Create an OAuth client](https://help.mypurecloud.com/?p=188023 "Goes to create an OAuth client page") in the Genesys Cloud Resource Center.
2. Add **http://localhost:3000** to the **Authorized redirect URIs**.

**Note**: If the **redirectUri** value has changed in the config file, you must add the new URI.

3. Add the following in the Scopes section
    * analytics
    * authorization
    * conversations
    * notifications
    * routing
    * users
4. Save the Client ID to use in the configuration project.

### Update configuration file

Modify the values in the configuration file before running the app. Use the values from the OAuth Client you created in the last step:

clientConfig.js:

```javascript
export const clientConfig = {
  GENESYS_CLOUD_CLIENT_ID: '<YOUR CLIENT ID HERE>',
  REDIRECT_URI: '<YOUR PRODUCTION URI HERE>',
};
```
### Run the app

Open a terminal and set the working directory to the root directory of the project, then run the following

```bash
npm install
npm run start
```

### Install and activate the Client Application in Genesys Cloud

1. Log in to your Genesys Cloud organization and add an integration. For more information see, [Add an integration](https://help.mypurecloud.com/articles/add-an-integration/ "Goes to Add an integration page") in the Genesys Cloud Resource Center.
2. Install the **Client Application** integration. For more information see, [Set up a Client Application integration](https://help.mypurecloud.com/articles/set-custom-client-application-integration/ "Goes to Set up a Client Application integration page") in the Genesys Cloud Resource Center.
3. (Optional) Use the Name box to give the widget a meaningful name (e.g., **Active Conversation Dashboard**).

![Client Application Integration](images/integration.png)

4. Click the Configuration tab.
5. In the Application URL box, type the URL of the web application. Be sure to specify the full URL.
`https://localhost:3000`
6. In the Application Type dropdown, select **widget**
7. To limit access to specific groups of agents, in Group Filtering, select the groups that use the widget.

![Client Application Integration Config](images/integration-config.png)

8. Activate the Client Application integration

### Test the solution

1. Set up a test queue with only you as a member since this guarantees you are assigned inbound calls to the queue. For more information see, [Create and configure queues](https://help.mypurecloud.com/?p=18650 "Goes to the Create and configure queues page") in the Genesys Cloud Resource Center.
** - Make sure that “Voice Transcription” is enabled in both queue settings, Speech, and Text Analytics

![Transcription Setting Queue](images/transcription-queue.png)
![Transcription Setting Analytics](images/transcription-speech-and-text.png)


2. Ensure there is an inbound call flow configured to transfer inbound calls to the selected queue. For more information see, [Work with inbound flows](https://help.mypurecloud.com/articles/work-with-inbound-call-flows/ "Goes to the Work with inbound flows page") in the Genesys Cloud Resource Center.

![Inbound Call Flow](images/inbound-call-flow.png)

3. Ensure there is a call route assigned to the inbound call flow from the previous step. For more information see, [Add a call route](https://help.mypurecloud.com/articles/add-a-call-route/ "Goes to the Add a call route page") in the Genesys Cloud Resource Center.

![Call route](images/call-route.png)

4. Ensure there is a DID number assigned to the call route from the previous step. For more information see, [Manage DID and toll-free number assignments](https://help.mypurecloud.com/?p=45223 "Goes to the Manage DID and toll-free number assignments page") in the Genesys Cloud Resource Center.

![DID Assignment](images/did-assignment.png)

5. To go **On Queue**, click the slider in the upper right-hand corner of the Genesys Cloud UI.
6. Using a phone, call (or have someone else call) the DID number mentioned above.
7. Use the dial pad or voice to answer the prompts from the IVR flow to navigate to your selected Queue.
8. Answer the incoming call in the Genesys Cloud app.
9. Open the Apps in the side navigation bar.
10. Open the client Application set up in the previous steps.
11. Find your queue in the **Active Conversation Dashboard** and expand the list to find your active conversations.

## Sample App overview

![Active conversation dashboard](images/embedded-app-title.png "Active conversation dashboard")

### Genesys Cloud Utils

The `src/utils/genesysCloudUtils.ts` file contains the intermediate functions that call Genesys Cloud SDK methods. These functions return promises that are handled upon resolution in the file and the invoking components themselves.

### Active conversation dashboard

This is the top level of the SPA (Single Page Application). The top-level consists of a tile, description, and a list of the queues in the logged-in user's organization.

### Queue listing

Each queue listing consists of a menu, and before the expansion, it shows the title and the number of active conversations in the queue. After the expansion, it displays the conversation listings for the queue.

### Conversation listing

![Conversation listing](images/embedded-app-in-action.png "Conversation listing")

Each conversation listing is a menu that expands and shows the conversation start time, the "standing" of the conversation (as defined in the summary of this document), the agent assigned to the conversation, and a live transcript.

## Configure the React project to use Genesys Cloud SDK

To integrate the Genesys Cloud SDK into your own React app, complete the following steps.

### Create a React project

If you are creating an app from scratch, run the following commands in a terminal in the directory of your choice:

```bash
npm install -g npx
npx create-react-app name-of-your-app --template TypeScript
```
If you configure an existing React app, you should use a version greater than v16.0 since the sample app uses React hooks introduced in React v16.0. See the tsconfig.json file in the root directory of this project for a TypeScript configuration example.

### Install NPM packages

1. Install the Genesys Cloud Platform Client

    ```bash
    npm install purecloud-platform-client-v2
    ```

### Import the platform-client-sdk to your project

Use the following process to import the platform-client-sdk

```javascript
const platformClient = require('purecloud-platform-client-v2/dist/node/purecloud-platform-client-v2.js');
```
Now, you can use the various API tools in the platformClient object.

Example

```javascript
const platformClient = require('purecloud-platform-client-v2/dist/node/purecloud-platform-client-v2.js');
const searchApi = new platformClient.SearchApi();
const usersApi = new platformClient.UsersApi();
const analyticsApi = new platformClient.AnalyticsApi();
const tokensApi = new platformClient.TokensApi();
const routingApi = new platformClient.RoutingApi();
const presenceApi = new platformClient.PresenceApi();
const conversationsApi = new platformClient.ConversationsApi();
```

## Additional resources

* [Genesys Cloud Platform SDK - JavaScript](https://developer.genesys.cloud/api/rest/client-libraries/javascript/)
* [GitHub repository](https://github.com/GenesysCloudBlueprints/partial-transcription-blueprint.git)
* [Create a new React app](https://reactjs.org/docs/create-a-new-react-app.html)

