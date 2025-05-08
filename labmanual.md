## NOTES for lab testers (Updated May 8, 2025, 1:10PM EDT)
> [!Alert] We are currently working through the following issues:
>
> - Computer use tools cannot be built until eng finishes allowlisting the environments
> - AIB actions in agent flows are currently failing, there is an ICM on this and it should be resolved quickly
> - The ability to create agent flows directly from the agent is expected to be live on Sunday per Heather. Instructions will be edited to accommodate this new flow, and that is when screenshots will be added to instructions.
> - Please ping matthewlehew with any issues encountered during testing.

# Extend agents everywhere: Agent flows and computer use tools in Copilot Studio

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
> Knowledge is a great tool for agents when you need to provide a lot of contextual information for the agent to consult at will. In this example, we want the agent to run the agent flow every time the email trigger is invoked, which can be handled with instructions alone. However, if we wanted to enable this agent for conversations about the company's invoice process or task it with making more subjective judgments over an invoice's validity, adding Knowledge would be the way to go.

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
> There are a couple reasons we are relying on the Get Attachment action rather than passing the attachment to the agent flow directly. First and foremost is security: In the event the agent is eventually designed to take on multiple tasks (or even conversations), we want to ensure invoice content can't accidentally be passed to the wrong place. With this design, the message and attachment IDs are being passed to the agent flow. Even if those IDs were passed to the wrong place, they can't be used to pull the attachment without the correct user authentication.
>
> The other relevant reason is simpler: today, passing objects from agents to agents flows isn't yet supported. :) 

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
> In our scenario, we have the purchase orders sitting in a specified document library, and the name of each purchase order is the same as the purchase order number. The expression we put in place for the file name appends the directory information to the SharePoint site address, adds the purchase order number, and then adds ".pdf" at the end. This lets the action get the content of the PDF.

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
> By default, an AI prompt action will output its response to a single text field. We want the action to output two separate fields: one for the decision and one for the reasoning behind the decision. Switching to JSON output lets us enable that, and pasting an example of what a JSON output would look like lets us define that schema for two values, a boolean called InvoiceMatch and a string called Reasoning.

- [] Click **Save** to save the prompt.

> [!Knowledge] At this point it would normally be a great idea to test the prompt by itself, but in the interest of time we'll push along.

We're almost done. Let's connect the pipes that will bring our invoice and purchase order to the prompt and then add the final action to respond to the agent.

- [] In the **Run a prompt** parameters, you'll see two new parameters corresponding to the two input values you added to the prompt.
  - [] For **Invoice**, type **/**, select **Insert dynamic content**, scroll down to the **Get Attachment (V2)** action in the dynamic field list, and select **Content Bytes**.
  - [] For **Purchase order**, type **/**, select **Insert dynamic content**, and select **File content** under **Get file content using path**.
- [] Click **+** to add a final action and search for +++Respond to the agent+++. Select **Respond to the agent** under the **Skills** connector.
- [] In the panel for the response action, select **Add an output**. Select the blue **Yes/No** option.
  - [] Replace "Enter a name" with +++InvoiceMatch+++.
  - [] Replace "Enter a value to respond with" with by typing **/**, selecting **Insert dynamic content**, and selecting **InvoiceMatch** under **Run a prompt**.
  - [] Replace "Enter a description of the output" with +++Whether the invoice matches the corresponding purchase order.+++
- [] Select **Add an output** again and select the purple **Text** option.
  - [] Replace "Enter a name" with +++Reasoning+++.
  - [] Replace "Enter a value to respond with" by typing **/**, selecting **Insert dynamic content**, and selecting **Reasoning** under **Run a prompt**.
  - [] Replace "Enter a description of the output" with +++The reason the model made its decision.+++
- [] Near the upper right corner of the designer, select **Save draft**, then **Publish**. Disregard the warning about **Run a prompt** not having a content approval action after it.

> [!Knowledge] On content approvals
>
> We are skipping the content approval because this is a controlled lab environment, but human review is always an important design element of a business process, especially processes augmented by AI. Even if it isn't part of the specific flow you build, your business process should include human review of important decisions or content generated by AI.

- [] We want to give the agent flow a presentable name, so let's click the **Overview** tab at the top, then click **Edit** in the upper right hand corner of the **Details** pane, then name the flow +++Invoice validation flow+++. Click **Save**.

Now that the agent flow is built, it's time to add it to the agent, set up the trigger, and test it out!

===

# Section 3: Adding the agent flow to the agent

- [] In the left hand nav, click **Agents**, then click **Invoice handling agent**.
- [] Scroll down to the **Actions** panel and click **Add action**. In the list of available actions that appears, you should see your newly-created flow. Click **Invoice validation flow** to start adding it.
- [] In the configuration window that appears, leave **Name** as it is and replace the content of **Description for the agent to know when to use this action** with +++Use this to determine whether a submitted vendor invoice matches the PO on file.+++
- [] If you like, you can expand the **Inputs and outputs** section to see the two inputs we added to the agent flow trigger appear here. We're going to trust the agent to fill those with the info we provide to the agent via its trigger.
- [] Click **Add action** to finish adding it to the agent. You should see it listed in the **Actions** panel.

# Section 4: Adding the trigger to the agent

We want this particular agent to run in response to a business event, not a conversation. To do that, we need to add a trigger to the agent. In just a couple steps, we are going to add (and customize) a trigger that will fire whenever an email arrives in your inbox.

- [] In the agent's **Overview** page, in the **Triggers** panel, select **Add trigger**.
- [] In the window that appears, select the trigger called **When a new email arrives (V3)** from the **Office 365 Outlook** connector. Click **Next**.
- [] On the next screen, you should see the connections automatically form. You can leave the trigger name as it is and click **Next**.
- [] On this screen, you can see the trigger's configuration options. Scroll to the bottom until you see the parameter called **Additional instructions to the agent when it's invoked by this trigger**. Erase what is in the field and replace it with +++An email with an invoice has arrived. Use @{triggerOutputs()?['body/value'][0]['id']} for Message ID and use @{triggerOutputs()?['body/value'][0]['attachments'][0]['id']} for Attachment ID to validate.+++

> [+Help] What did that do? (Optional info)
>
> By default, the **When a new email arrives (V3)** trigger passes all of the email content to the agent. For the security reasons we mentioned earlier in this lab, this is an example of more tightly controlling that flow of information, only providing the agent with the Message and Attachment ID. This also simplifies the agent's task by giving it less information to parse. This step can often be skipped, but if you're experiencing unpredictable agent behavior, it's always a good idea to take a look at the flows of information to and from the agent.

- [] Click **Create trigger**, then click **Close**.
- [] On the top of the agent overview page, click **Publish**, then click **Publish** again.
- [] We want to ensure the agent can connect to the flow, so in the ellipsis menu next to **Test your agent**, select **Manage connections**. In the new tab that opens, in the row that says **Invoice validation flow**, click **Connect**, verify the connections have a green check mark, then click **Submit**. Once the status is Connected, you can close that tab to return to Copilot Studio.

# Section 5: Time to test!

Let's see what we've got so far! Open a new tab in Edge and visit +++https://forms.office.com/r/N41p1fHAin+++. Selecting **Compliant** will randomly send one of four invoices that matches its corresponding purchase order in SharePoint. Selecting **Not compliant** will send an invoice that has the correct items and the correct total amount, but the quantities and prices of the items are different.

- [] Select **Compliant** and submit the form, then return to your Copilot Studio tab.
- [] Under the **Activity** tab, you should be able to see the agent run that just started. Click on it, and you can see how the agent successfully called the agent flow and received the response.

# Section 6: Computer use

The last part of this lab involves adding a computer use tool so the agent can log the compliant invoice into the web app.
- [] In the Overview tab of the newly created agent, scroll down to **Actions**, click on **Add action**, **New action** and select **Computer use**.
- [] In the first step you will be asked to create a connection to Computer use. Select the machine that has been provisioned. For username and password, enter the ones to connect to the machine.
- [] After the connection's details have been confirmed, provide the following information to set up the tool:
    * **Name:** +++Register new invoices+++
    * **Description:** +++Register new invoices from the last 24h+++
    * **Instructions:** +++Go to https://rpxpmshare.blob.core.windows.net/web/Contoso/invoice-manager.html, filter invoices by the last 24 hours, open the invoice PDF and using its content create an invoice at https://rpxpmshare.blob.core.windows.net/web/Contoso/index.html+++

Once you have configured the computer use action, you can click on it to see its details and test it. When you click on test, computer use will take a few seconds to connect to the machine. Then you will see on the left the reasoning steps from computer use and on the right the preview of what is happening in the machine. 

When the computer use action is used by your agent, you can go to Activity. Select an agent run, and when switching to the Transcript view you will be able to see the full activity of what your computer use tool did.
