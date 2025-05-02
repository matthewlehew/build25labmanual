@lab.Title

## Lab Goals

Enter your ID number here to use as part of your login later, this will help save time and reduce the risk of errors:

@lab.TextBox(IDnumber)

Your username is +++user@lab.Variable(IDnumber)@build2025automations.onmicrosoft.com+++.

Autonomous agents empower makers to solve complex problems, combining advanced reasoning capabilities with powerful tools. Learn how agent flows (whether built as a standalone workflow or as a tool for an agent to call) and UI automation through computer use tools can easily be built and deployed as part of a comprehensive business process transformation effort. Learn how to keep your information secure, help an agent know exactly when to use the right tool, and more—all within Copilot Studio.

The goal of this lab is to give your agent the ability to assign tasks to your coworkers by building an automation that creates tasks in Microsoft Planner, assigns them to the right people, and sends them a notice in Microsoft Teams. 

# Section 1: Logging in and creating an agent flow

> [!Knowledge] Text formatted as an +++example+++ represents type text. Clicking on this text will automatically insert it to prevent any typing errors.

> [!Alert] Make sure you change environments!

> [+Help] Helper text
>
> This is a test of expandable help blocks.

* To begin, log into the virtual machine using the following credentials: +++@lab.VirtualMachine(Win11-Pro-Base).Password+++
* Open Edge and navigate to copilotstudio.microsoft.com
* To sign in, use the number you were given as you walked into the session to replace the **?**’s in the username.
  * Username: user???@build2025automations.onmicrosoft.com
  * Password: +++test@Build25.Password+++
* Click “Get Started” to be brought to an agent building experience, **but don’t start building the agent yet!**
* In the upper right corner, click the environment name **Build 25 (default)** and change it to the Developer environment named after your user id.
* You will land on the Copilot Studio home page in the new environment. If you see a “Welcome to Copilot Studio” banner, click Skip.
* In the left hand nav, click **Flows**.
* Select **+New flow++ to create a new agent flow.

Now it's time to use natural language to quickly build a new agent flow.

* Describe your flow to Copilot using natural language with the following prompt: +++Using the button trigger, when a user manually uploads an invoice file, save the file to OneDrive folder and send a notification to a Teams channel.+++
* Click the arrow button to start the building process.
* A suggested flow will appear. Read through it and select "Keep it and continue."
* Because this is a new environment, you'll have to set up connections to the necessary apps and services. This should happen automatically, so once a green check appears, select Create.

After landing in the designer with your agent flow, it's time to make a few more adjustments.

* To set up the trigger to accept an uploaded file, start by clicking the **Manually trigger a flow** trigger. Click **Add an input** and select the **File** option. 
* Next, select the **Create file** OneDrive action that is under the trigger. We want to make sure the file saves as a PDF, so add +++.pdf+++ to the end of the dynamic file name that's already listed in the **File name** field. 

We are going to add two more actions to the flow, which will extract information from the invoice and use AI to generate a summary.

* Below the **Create file** action, select the **+** icon. 
* In the **Add action** menu, search for +++Extract information from invoices+++ and add it to the flow. 
* The Dataverse connection should happen automatically in a brief popup. 
* Under the parameters for this new action, navigate to the **Invoice File** parameter. 
* Click the field and type a "/" to show the available dynamic content to add. 
* Search for +++File Content ContentBytes+++, then click on the parameter that appears under the **Manually trigger a flow** action.
* Now, add a new action by selecting the **+** icon below the **Extract information from invoices** action.
* Search for +++Create text with GPT using a prompt+++ and add it.
* Under the parameters for this new action, navigate to the **Prompt** parameter. Click the dropdown and select **AI summarize**.
* Under the **Input text** parameter that appears, add the following. You will also have to update the dynamic tokens. +++Generate a concise summary that we have received an invoice include a summary of the following details <vendor name>, <amount due (text)>, and <Due date (text)>.

Finally, we are going to configure the "Post a Message" Teams action.

* Click the "Post a message" action and update it with the following parameter values.
* Post as: select **Flow bot**
* Post in: select **Chat with Flow bot**
* Recipient: select yourself as the recipient.
* Message: Open the dynamic content panel by clicking the message field and select **Text** from the dynamic tokens provided under the **Create text with GPT using a prompt** action.

It's now time to save and test the flow.

* Click **Save draft** and then **Publish**.
* Test the flow by selecting *Test*, then uploading the invoice PDF through the trigger.
* **Note:** You may see a warning about not having a content approval action in your flow. Since this is a demo, we have omitted it. However, for production scenarios requiring human review, adding a content approval action is highly recommended.

# Section 2: Adding computer use to an agent.

* From the Copilot Studio home page, click Create in the left navigation menu, then click New agent.

**Note: You can experiment with the conversational building process if you’d like, or you can follow the bullet points to quickly move forward.**

* Click **Skip to configure** in the upper right corner.
* Name the agent +++Invoice processing agent+++
* In the description, add: +++An agent to help with managing invoice processing.+++
* In the instructions, add: +++You are an agent meant to assist teams with handling and tracking invoices. Always be respectful and courteous.+++
* Click **Create**.
* In the Overview tab of the newly created agent, look for the **Orchestration** toggle and turn it on.
* Scroll down to **Actions**, click on **Add action**, **New action** and select **Computer use**.
* In the first step you will be asked to create a connection to Computer use. Select the machine that has been provisioned. For username and password, enter the ones to connect to the machine.
* After the connection's details have been confirmed, provide the following information to set up the tool:
    * **Name:** +++Register new invoices+++
    * **Description:** +++Register new invoices from the last 24h+++
    * **Instructions:** +++Go to https://rpxpmshare.blob.core.windows.net/web/Contoso/invoice-manager.html, filter invoices by the last 24 hours, open the invoice PDF and using its content create an invoice at https://rpxpmshare.blob.core.windows.net/web/Contoso/index.html+++
* Once you have configured the computer use action, you can click on it to see its details and test it. When you click on test, computer use will take a few seconds to connect to the machine. Then you will see on the left the reasoning steps from computer use and on the right the preview of what is happening in the machine. 
*  When the computer use action is used by your agent, you can go to Activity. Select an agent run, and when switching to the Transcript view you will be able to see the full activity of what your computer use tool did.
