# Extend agents everywhere: Agent flows and computer use tools in Copilot Studio

## Setup step

> [!Alert] Do not skip this step! It will affect some of the login info that appears later in these instructions.

Enter your three-digit number you were given in this box. It will be used as part of your login later, which will help save time and reduce the risk of errors. 

@lab.TextBox(IDnumber)

If you see the number appear here, you're good to go: @lab.Variable(IDnumber)

## Lab Goals

The goal of this lab is to build an agent that utilizes agent flows and computer use to process incoming invoices. In this scenario, vendors upload invoices to a web portal that does **not** provide API access, and a copy of the invoice is sent to a SharePoint document library. You are wanting to:

- Use AI to determine if the invoice aligns with the corresponding purchase order (also hosted in a SharePoint document library)
- Log invoices into a payment authorization platform that, like the web portal the vendors use, doesn't provide API access.

Let's get started!

# Section 1: Logging in and creating an agent

> [!Knowledge] Text formatted as an +++example+++ represents type text. Clicking on this text will automatically insert it wherever your lab cursor is. Use this to prevent any typing or copy/pasting errors!

- [] To begin, log into the virtual machine with this password: +++@lab.VirtualMachine(Win11-Pro-Base).Password+++
  - [] If you see a full-page splash about backing up your PC, click **Opt-out of backup** and then **Skip for now**.
- [] Open Edge and navigate to +++make.powerautomate.com+++.
- [] To sign in, use the following credentials. Choose **Yes** on the dialog asking if you want to stay signed in, and dismiss any prompts from Edge that offer to reuse the password.
  - Username: +++user@lab.Variable(IDnumber)@build2025automations.onmicrosoft.com+++
  - Password: +++test@Build25+++

> [!Alert] **Important:** Make sure you let the Power Automate home page load fully. Because it's the user account's first login, it may take a while. You should see the environment name and user account information appear in the upper-right hand corner before you navigate away.

- [] In the left nav, at or near the bottom, click **Power Platform** (Depending on your screen resolution, you may just see the Power Platform logo), then **Copilot Studio**.

!IMAGE[Img1.png](instructions291722/Img1.png)

> [+Help] Why did I start in Power Automate? (Optional info)
>
> There's a bug in test environments regarding brand new accounts like the lab accounts you're using today. This is a little trick to avoid it before it's fixed. :)

- [] Click **Get Started** on the welcome dialog if it appears (and then **Skip** on the next welcome dialog if it appears) to be brought to an agent building experience, **but don’t start building the agent yet!**

> [!Alert] Make sure you follow the next step and change your environment!

- [] In the upper right corner, click the environment name **Build2025Automations (default)** and click the environment named **user@lab.Variable(IDnumber)**, which is the Developer environment named after your user id, listed under **Supported environments**.

!IMAGE[Img2.png](instructions291722/Img2.png)

- [] You will land on the Copilot Studio home page in the developer environment. If you see a “Welcome to Copilot Studio” banner, click **Next** twice, then **Done**.
- [] In the left hand nav, click **Create**, then click **New agent**.

> [!Knowledge] You can experiment with the natural language process if you'd like, but we recommend following these steps to move quickly:

- [] Click the **Configure** in the sliding tab control on the left hand side, then fill the following fields:
  - Name: +++Invoice handling agent+++
  - Description: +++An agent that validates and logs incoming vendor invoices.+++
  - Instructions: +++When an invoice arrives via email, send the attachment information to the invoice validator agent flow to see if it's compliant. If it is, log the invoice in the payment system.+++
- [] Click **Continue** in the upper right corner to land on the agent's Overview page. Under Details > Orchestration, the toggle should default to **Enabled**. If it doesn't, turn the toggle on.

> [+Help] Why didn't we add knowledge? (Optional info)
>
> Knowledge is a great tool for agents when you need to provide a lot of contextual information for the agent to consult at will. In this example, we want the agent to run the agent flow every time the email trigger is invoked, which can be handled with instructions alone. However, if we wanted to enable this agent for conversations about the company's invoice process or task it with making more subjective judgments over an invoice's validity, adding Knowledge would be the way to go.

Now it's time to create our agent flow, starting on the next page.

===

# Section 2: Creating the agent flow

- [] We'll add the flow directly from this agent. Scroll down to the **Tools** panel and click **Add tool**.
- [] From this **Add tool** surface, click **New tool**, then select **Agent flow**.

You'll land in the designer with two actions already created. The first is the trigger that accepts the call from the agent, and the second is the action that sends a response. The trigger will always be at the top, and the response will usually need to stay at the bottom.

- [] Click the **When an agent calls the flow** trigger. In the panel that appears, click **Add an input**, then the purple **Text** option.
  - [] In the name field, replace "Input" with +++File path+++.
  - [] In the description field, replace "Please enter your input" with +++Use the file path of the new invoice.+++

> [+Help] What's up with the input variables? (Optional info)
>
> There are a couple reasons we are passing a path to the agent flow rather than passing the file content directly. First and foremost is security: In the event the agent is eventually designed to take on multiple tasks (or even conversations), we want to ensure invoice content can't accidentally be passed to the wrong place. With this design, only the file metadata, not the content, is handled by the agent. Even if the file path were passed to the wrong place, it couldn't be used to pull the content without the correct SharePoint access privileges.
>
> The other relevant reason is simpler: today, passing objects from agents to agents flows isn't yet supported. :) 

- [] In the main canvas of the designer, click the **+** icon below the trigger (in between the two actions), then search for +++Get file content using path+++ and select the **Get file content using path** action that appears under **SharePoint**.
  - [] Click **Sign in** to create your user's connection to SharePoint. **Do not check** "Connect via on-premises data gateway." Select your user account in the pop-up window. If asked, click **Allow access**.
  - [] If you see an OAuth connection error, click the **Pop-Up Block** icon on the right side of the Edge address bar, then select "Always allow pop-ups and redirects from..." and click **Done**. Then try clicking **Sign in** again.

!IMAGE[Img3.png](instructions291722/Img3.png)

- [] Fill the two required parameters for the **Get file content** action:
  - [] In the **Site Address** parameter, enter +++https://build2025automations.sharepoint.com/sites/FinanceCentral+++ and select **"Use https://build2025automations.sharepoint.com/sites/FinanceCentral as a custom value"** in the dropdown.
  - [] In the **File Path** parameter, type a **/** to bring up the insert menu, then select **Insert dynamic content**. In the list of parameters that appears, under **When an agent calls the flow**, click **File path**. This will insert a dynamic tag in the parameter field.

It's time to add an AI-powered action that will pull the purchase order number from the invoice, which will help us grab the correct PO from SharePoint.

- [] Click the **+** button under **Get file content using path** on the canvas (directly above **Respond to the agent**) to add a new action, then search for +++Process invoices+++ and click the **Process invoices** action under **AI Builder**.
  - [] Click **Sign in** and create the connection.
  - [] In the **Invoice file** parameter, type **/** and click **Insert dynamic content**. Select **File Content** under the **Get file content using path** action in the parameter list.
- [] Click the **+** button below **Process invoices** (again, directly above the **Respond to the agent** action) to add an action after the invoice processing action. Search for +++Get file content using path+++ and click the **Get file content using path** action under **SharePoint** to add a *second* instance of this action, which will be renamed **Get file content using path 1** in the designer.
  - [] In the **Site Address** parameter of the new action (make sure you're editing the one named **Get file content using path 1**), enter +++https://build2025automations.sharepoint.com/sites/FinanceCentral+++ and select **"Use https://build2025automations.sharepoint.com/sites/FinanceCentral as a custom value"** in the dropdown.
  - [] Automatic typing won't work for this step, so copy the following text by clicking it once, then paste it into the **File Path** field: ++@{concat('/Purchase Orders/',outputs('Process_invoices')?['body/responsev2/predictionOutput/result/fields/purchaseOrder/valueText'],'.pdf')}++
  - [ ] Verify the preceding step was successful by checking to make sure an expression token was pasted into the field.

!IMAGE[Img4.png](instructions291722/Img4.png)

> [+Help] What is this doing? (Optional info)
>
> In our scenario, we have the purchase orders sitting in a specified document library, and the name of each purchase order is the same as the purchase order number. The expression we put in place for the file name appends the directory information to the SharePoint site address, adds the purchase order number, and then adds ".pdf" at the end. This lets the action get the content of the PDF.

We have the file content for our vendor invoice and the corresponding purchase order, now it's time to make sure they align!

- [] Click the **+** button under **Get file content using path 1** (the one right above **Respond to the agent**) to add a new action, then search for +++Run a prompt+++. Select the **Run a prompt** button under AI Builder, then, if asked, click the existing Microsoft Dataverse connection that was created earlier.
- [] For the **Prompt** parameter, click the dropdown menu and select **New custom prompt**.
- [] After dismissing any teaching popups that appear, add +++Invoice validation prompt+++ as the prompt title at the top of the modal that appeared.
- [] In the **Instructions** field, paste +++Compare the content of the invoice to the content of the purchase order. Make sure the items ordered and amount charged both align. Invoice: (Invoice content) Purchase order: (Purchase order content)+++ into the field.
  - [] Remove "(Invoice content)" from the prompt, place the cursor after "Invoice:", then type a **/** to open the add content menu. Select **Image or document** and then select **Ok** when the dialog appears about changing the model to GPT-4o. Rename the input from "Document input" to +++Invoice+++ and select **Close** (you can skip the sample data upload).
  - [] Remove "(Purchase order content)" from the prompt, place the cursor after "Purchase order:", then type a **/** to open the add content menu again. Select **Image or document** and then rename the input from "Document input" to +++Purchase order+++. Select **Close**.
 
We're almost done with the prompt. Now we want to make sure it outputs the response in a format we can work with (as opposed to free text). 

- [] On the right side of the prompt workspace, in the upper right hand corner of the **Model response** panel, click the **Text** dropdown and change the output format to **JSON**. In the same area of the screen, click the **Output settings** icon to the left of the word **Output**.

!IMAGE[Img5.png](instructions291722/Img5.png)

- [] The left side of the panel has changed to a JSON configuration experience. In the new window, paste the following: +++{"InvoiceMatch": true, "Reasoning": "The details of the invoice match the details of the purchase order."}+++ Then click **Apply**.

!IMAGE[Img6.png](instructions291722/Img6.png)

> [+Help] What was that about? (Optional info)
>
> By default, an AI prompt action will output its response to a single text field. We want the action to output two separate fields: one for the decision and one for the reasoning behind the decision. Switching to JSON output lets us enable that, and pasting an example of what a JSON output would look like lets us define that schema for two values, a boolean called InvoiceMatch and a string called Reasoning.

- [] Click **Save** to save the prompt.

> [!Knowledge] At this point it would normally be a great idea to test the prompt by itself, but in the interest of time we'll push along.

We're almost done. Let's connect the pipes that will bring our invoice and purchase order to the prompt and then configure the final action to respond to the agent.

- [] In the **Run a prompt** parameters, you'll see two new parameters corresponding to the two input values you added to the prompt.
  - [] For **Invoice**, type **/**, select **Insert dynamic content**, scroll down to find **File Content** under the **Get file content using path** action in the dynamic field list.
  - [] For **Purchase order**, type **/**, select **Insert dynamic content**, and select **File content** under **Get file content using path 1**. 
  
> [!Alert] You are bringing two different SharePoint files to the AI prompt. Make sure you are selecting the **File Content** parameter from the correct actions. The invoice is coming from **Get file content using path** and the purchase order is coming from **Get file content using path 1**.
>
> This type of ambiguity is best resolved by replacing the action names with something more clear once they're added to the designer, which would also help your collaborators better understand your flow, but for the purpose of this lab we are skipping that.

- [] Click the **Respond to the agent** action. In the parameter panel, select **Add an output**. Select the blue **Yes/No** option.
  - [] Replace "Enter a name" with +++InvoiceMatch+++.
  - [] Replace "Enter a value to respond with" with by typing **/**, selecting **Insert dynamic content**, and selecting **InvoiceMatch** under **Run a prompt**.
  - [] Replace "Enter a description of the output" with +++Whether the invoice matches the corresponding purchase order.+++
- [] Select **Add an output** again and select the purple **Text** option.
  - [] Replace "Enter a name" with +++Reasoning+++.
  - [] Replace "Enter a value to respond with" by typing **/**, selecting **Insert dynamic content**, and selecting **Reasoning** under **Run a prompt**.
  - [] Replace "Enter a description of the output" with +++The reason the model made its decision.+++

> [+Help] What's up with this response? (Optional info)
>
> Just as we created two input variables for the agent to pass to the flow, we are creating two output variables for the flow to pass back to the agent. The agent will be able to see whether the invoice matched the purchase order, as well as the reason the AI prompt chose the answer it did.

- [] Near the upper right corner of the designer, select **Save draft**, then **Publish**. Disregard the warning about **Run a prompt** not having a content approval action after it. Click **Go back to agent** on the dialog that appears.

!IMAGE[Img7.png](instructions291722/Img7.png)

> [!Knowledge] We are skipping the content approval because this is a controlled lab environment, but human review is always an important design element of a business process, especially processes augmented by AI. Even if it isn't part of the specific flow you build, your business process should include human review of important decisions or content generated by AI.

You'll land in the Tools page of the Invoice handling agent (it may take a few seconds for your flow to appear). Let's rename the agent flow and make sure the agent knows when to call it. 

- [] Click on the agent flow, which is currently called "Untitled." Enter +++Invoice validation flow+++ as the model display name and +++Use this to determine whether a submitted vendor invoice matches the PO on file.+++ as the description. Click **Save**.

> [+Help] What about the name of the agent flow itself? (Optional info)
>
> Future updates are streamlining the process of renaming your agent flow. For today, we've accomplished the important thing, which is setting the name and description the agent sees when determining what tool to use.

Now that the agent flow is built and added, it's time to set up the trigger and test it out, starting on the next page!

===

# Section 3: Adding the trigger to the agent

We want this particular agent to run in response to a business event, not a conversation. To do that, we need to add a trigger to the agent. In just a few steps, we are going to add (and configure) a trigger that will fire whenever a document is placed in the specified SharePoint library.

- [] In the agent's **Overview** page, scroll down to the **Triggers** panel and select **Add trigger**.
- [] In the window that appears, select the trigger called **When a file is created (properties only)** from the **SharePoint** connector. If it isn't visible, you can use the Search field to locate it. Click **Next**.
- [] On the next screen, after a loading period, you should see the connections automatically form. You can leave the trigger name as it is and click **Next**.
- [] On this screen, you can see the trigger's configuration options. For **Site Address**, click the dropdown and select **Add a custom item**. Enter +++https://build2025automations.sharepoint.com/sites/FinanceCentral+++ in the dialog that appears and select **OK**.
- [] For **Library Name**, select "Invoices" from the dropdown.
- [] For **Folder**, paste +++/Invoices/@lab.Variable(IDnumber)+++
- [] Click **Create trigger**, then click **Close** when you see a celebratory message saying it's time to test.
- [] On the top of the agent overview page, click **Publish**, then click **Publish** again.
- [] We want to ensure the agent testing context can connect to the flow, so in the three dot menu next to **Test your agent**, select **Manage connections**.

!IMAGE[Img8.png](instructions291722/Img8.png)

- [ ] In the new tab that opens, in the row that says **Invoice validation flow**, click **Connect**, verify the SharePoint connection has a green check mark, then click **Submit**. Once the status is Connected, you can close that tab to return to Copilot Studio.

# Section 4: Time to test!

Let's see what we've got so far! Open a new tab in Edge and visit +++https://forms.office.com/r/N41p1fHAin+++. Selecting **Compliant** will randomly deposit one of four invoices that matches its corresponding purchase order in SharePoint. Selecting **Not compliant** will send an invoice that has the correct items and the correct total amount, but the quantities and prices of the items are different.

- [] Select **Compliant** and submit the form, then return to your Copilot Studio tab.
- [] In the **Triggers** panel of the agent overview page, click the beaker icon in the same row as the **When a file is created (properties only)** trigger. This opens a dialog that will show the trigger instance you just initiated. By default, the SharePoint trigger checks for files every 60 seconds, so you may need to click the circular refresh arrow a few times to see the trigger instance appear.
- [] With the recent trigger instance selected, click **Start testing**. You'll see the Activity map, which shows how the the agent calls the agent flow, provides the file path as an input, and a few moments later sees the response values fill.

# Section 5: Computer use

> [!Alert] Computer use is a capability in **research preview** and is not quite ready for production primetime. The near-term roadmap includes many improvements, including robust, secure credential handling.
>
> For today's demo, we are streamlining the scenario with a test site that does not require authentication. The goal of this demo is to help you visualize how, with credential management, computer use could finish the agent's job of logging the validated invoice into a legacy system of record.

The last part of this lab involves adding a computer use tool so the agent can log the compliant invoice into the web app.
- [] In the Overview tab of the agent, scroll down to **Tools**, click **Add tool**, then click **New tool**, and then select **Computer use**.
- [] In the first step you will be asked to create a connection to the Computer use tool. Click the **Create** link and dismiss the "Switch to credentials" teaching bubble. 
- [] For **Agent machine**, click the dropdown and select **ComputerUse@lab.Variable(IDnumber)**. 
- [] For **Domain and username**, enter +++computeruser@build2025automations.onmicrosoft.com+++.
- [] For **Password**, enter +++test@Build25+++.
- [] Click **Create**, then **Add and configure**. 
- [] On the **Details** page, provide the following information to finish setting up the computer use tool:
  - [] **Name:** +++Register new invoices+++
  - [] **Description:** +++Register new invoices from the last 24h+++
  - [] **Instructions:** +++Go to https://rpxpmshare.blob.core.windows.net/web/Contoso/invoice-manager.html, filter invoices by the last 24 hours, open the invoice PDF and using its content create an invoice at https://rpxpmshare.blob.core.windows.net/web/Contoso/index.html+++
- [] Click **Save**.
- [] Click **Publish** in the upper right hand corner of the page to republish the agent.

Let's see it work!

- [] Click **Test** (right above the Instructions input) to see the computer use tool work. It will take a few seconds to connect to the machine. Then, you will see on the left the reasoning steps from the model. On the right you'll see the preview of what is happening on the machine.

!IMAGE[Img9.png](instructions291722/Img9.png)

> [!Knowledge] Notice how, after opening the second tab, the tool is able to successfully navigate the "unexpected" system maintenance notice, a very common issue that trips up traditional UI automations.

- [] When you go to the agent's **Activity** panel, you can click the latest run to see the activity record of the computer use tool. Clicking the **Activity map** dropdown and switching to **Transcript** will let you see the images that show how the tool progressed through its task.

!IMAGE[Img10.png](instructions291722/Img10.png)

Once you have configured the computer use action, you can click on it to see its details and test it. When you click on test, computer use will take a few seconds to connect to the machine. Then you will see on the left the reasoning steps from computer use and on the right the preview of what is happening in the machine. 

# Conclusion

Amazing conclusion will go here. 
