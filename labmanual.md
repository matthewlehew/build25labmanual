@lab.Title

## Setup step

> [!Alert] Do not skip this step! It will affect some of the login info that appears later in these instructions.

Enter your three-digit number you were given in this box. It will be used as part of your login later, which will help save time and reduce the risk of errors. 

@lab.TextBox(IDnumber)

If you see the number appear here, you're good to go: @lab.Variable(IDnumber)

## Lab Goals

The goal of this lab is to build an agent that utilizes agent flows and computer use to process  incoming invoices. In this scenario, customers upload invoices to a web portal that does **not** provide API access, and the portal emails you a copy of the invoice. You are wanting to:

- Use AI to determine if the invoice aligns with the corresponding purchase order (hosted in a SharePoint document library)
- Log invoices into a payment authorization platform that, like the web portal your customers use, doesn't provide API access.

Let's get started!

# Section 1: Logging in and creating an agent

> [!Knowledge] Text formatted as an +++example+++ represents type text. Clicking on this text will automatically insert it wherever your lab cursor is. Use this to prevent any typing or copy/pasting errors!

- [] To begin, log into the virtual machine with this password: +++@lab.VirtualMachine(Win11-Pro-Base).Password+++
- [] Click here to open Edge and navigate to +++https://copilotstudio.microsoft.com+++.
- [] To sign in, use the following credentials. Choose the option to stay signed in, and dismiss any prompts from Edge that offer to reuse the password.
  - Username: +++user@lab.Variable(IDnumber)@build2025automations.onmicrosoft.com+++
  - Password: +++test@Build25+++
- [] Click **Get Started** on the welcome dialog (and then **Skip** on the next welcome dialog if it appears) to be brought to an agent building experience, **but don’t start building the agent yet!**

> [!Alert] Make sure you follow the next step and change your environment!

- [] In the upper right corner, click the environment name **Build2025Automations (default)** and change it to the Developer environment named after your user id, which should be listed under **Supported environments**.
- [] You will land on the Copilot Studio home page in the developer environment. If you see a “Welcome to Copilot Studio” banner, click **Skip**.
- [] In the left hand nav, click **Create**, then click **New agent**.

> [!Knowledge] You can experiment with the natural language process if you'd like, but we recomment following these steps to move quickly:

- [] Click **Skip to configure** in the upper right corner, then fill the following fields:
  - Name: +++Invoice handling agent+++
  - Description: +++An agent that validates and logs incoming vendor invoices.+++
  - Instructions: +++When an invoice arrives via email, send the attachment information to the invoice validator agent flow to see if it's compliant. If it is, log the invoice in the payment system.+++
- [] Click **Create** to land on the agent's Overview page. Under Details > Orchestration, the toggle should appear grey for a few seconds and then default to **Enabled**. If it doesn't, turn the toggle on.

> [+Help] Why didn't we add knowledge? (Optional info)
>
> This is a test of expandable help blocks.

Now it's time to create our agent flow.

===

# Section 2: Creating the agent flow

- [] In the left hand nav, click **Flows**, and then click **Start in designer** in the landing page.
- [] In the designer, click **Add a trigger**, then type +++Copilot Studio+++ in the search field. Under **Skills**, click the trigger called **When an agent calls the flow.**
- [] In the panel that appears, click **Add an input**, then the purple **Text** option.
  - [] In the name field, replace "Input" with +++Message ID+++.
  - [] In the description field, replace "Please enter your input" with +++Use the email's Message ID.+++
- [] Click **Add an input** again and select the purple **Text** option again.
  - [] In the name field, replace "Input 1" with +++Attachment ID+++.
  - [] In the description field, replace "Please enter your input" with +++Use the email's Attachment ID.+++
- [] In the main canvas of the designer, click the **+** icon below the trigger, then search for +++Get Attachment+++ and select the **Get Attachment (V2)** action that appears under **Office 365 Outlook**.
  - [] Click **Sign in** to create your user's connection to Outlook. Select your user account in the pop-up window. If asked, click **Allow access**.
  - [] If you see an OAuth connection error, click the **Pop-Up Block** icon on the right side of the Edge address bar, then select "Always allow pop-ups and redirects from..." and click **Done**. Then try clicking **Sign in** again.
- [] Fill the two required parameters for the Get Attachment action:
  - [] In the Message Id parameter, type a **/** to bring up the insert menu, then select **Insert dynamic content**. In the list of parameters that appears, under **When an agent calls the flow**, click **Message ID**. This will insert a dynamic tag in the parameter field.
  - [] Do the same thing in the Attachment Id parameter, except under **When an agent calls the flow**, click **Attachment ID**.

> [+Help] Why are we doing it this way? (Optional info)
>
> This is a test of expandable help blocks.

It's time to add an AI-powered action that will pull the purchase order number from the invoice, which will help us grab the correct PO from SharePoint.

- [] Click the **+** button on the canvas to add a new action, then search for +++Extract information from invoices+++ and click the **Extract information from invoices** action under **AI Builder**.
  - [] Click **Sign in** and create the connection.
  - [] In the **Invoice file** parameter, type **/** and click **Insert dynamic content**. Select **Content Bytes** under the **Get Attachment (V2)** action in the parameter list.
- [] Click the **+** again to add an action after the invoice extraction action. Search for +++Get file content using path+++ and click the **Get file content using path** action under **SharePoint**.
  - [] Create the connection. **Do not check** "Connect via on-premises data gateway"
  - [] In the **Site Address** parameter, enter +++https://build2025automations.sharepoint.com/sites/FinanceCentral+++ and select **"Use https://build2025automations.sharepoint.com/sites/FinanceCentral as a custom value"** in the dropdown.
  - [] In the **File Path** parameter, type **/** and click **Insert expression**, which is the first time you're clicking this particular option.
  - [] In the expression editor flyout that appears, paste +++concat('/Purchase Orders/',outputs('Extract_information_from_invoices')?['body/responsev2/predictionOutput/result/fields/purchaseOrder/valueText'],'.pdf')+++ and then click **Add**.

> [+Help] What is this doing? (Optional info)
>
> This is a test of expandable help blocks.

We have the file content for our vendor invoice and the corresponding purchase order, now it's time to make sure they align!

- [] Click the **+** button to add a new action, then search for +++Run a prompt+++. Select the **Run a prompt** button under AI Builder.
- [] For the **Prompt** parameter, click the dropdown menu and select **New custom prompt**.
- [] After dismissing any teaching popups that appear, add +++Invoice validation prompt+++ as the prompt title.
- [] In the instructions field, paste the following: +++Compare the content of the invoice to the content of the purchase order. Make sure the items ordered and amount charged both align. Invoice: 
 Purchase order: +++
  - [] Place the cursor after "Invoice:" in the prompt, then click **Add content** at the bottom of the instructions window. Select **Image or document** and then select **Ok** when the dialog appears about changing the model to GPT-4o. Rename the input to **Invoice** and select **Close** (you can skip the sample data upload).
  - [] Place the cursor after "Purchase order:" in the prompt, then click **Add content** again. Select **Image or document** and then name the input **Purchase order**. Select **Close**.
 
We're almost done with the prompt. Now we want to make sure it outputs the response in a format we can work with (as opposed to free text). 

- [] On the right side of the prompt workspace, in the upper right hand corner of the **Model response** panel, click the **Text** dropdown and change the output format to **JSON**. In the same area of the screen, click the **Output settings** icon to the left of the word **Output**.
- [] The left side of the panel has changed to a JSON configuration experience. In the new window, paste the following: +++{"InvoiceMatch": true, "Reasoning": "The details of the invoice match the details of the purchase order."}+++ Then click **Apply**.

> [+Help] What was that about? (Optional info)
>
> This is a test of expandable help blocks.

- [] 



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
