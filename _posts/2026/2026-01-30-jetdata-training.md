---
title: "JETData.AI Training - Advance 1"
date: 2026-02-4 02:54:00 +0700
categories: [Training]
tags: [AI, AI Agent, N8N, API]
image: /assets/img/posts/jetdata-training/banner.png
alt: "Final Assignment Write Up"
description: "Final Assignment Write Up for Completing JETData.AI - Advance 1 Training."
pin: false
---

## **What is JETData.AI and n8n?**
> Check out [JETData.AI](https://jetdata.ai/) and [n8n](https://n8n.io/)
{: .prompt-tip}

`JETData.AI` is a platform that helps business manage, store, and use their data easily, all in one place. What makes it great? JETData.AI comes with built in automation and AI on their platform, so with stuff like that, can help with automating the manual process and hopefully minimize human error.

`n8n` is a tool to automate tasks by connecting several nodes/apps together. By using n8n we can build a fully working workflow.

---

## **Why I Make This Posts?**
Hopefully by creating this post, people that see this and facing the same problem as me, can find some inspiration on how they might mitigate or at least minimize the issues they are experiencing.

Through this post, I also want to share the real results of the training.

---

## JETData.AI Basics
A Quick explanation for JETData.AI Basics UI Navigation.

### Login Page & Main Page

![Login Page](/assets/img/posts/jetdata-training/jetdata-loginpage.png){: width="500" height="400"}
_JETData.AI Login Page_


You can login to JETData.AI using Google or Microsoft account, but I will login with Username, Password, and Site ID.

What is Site ID?, Site ID is a identification for login with your username and password, it can be like company name, or anything.

![Main Page](/assets/img/posts/jetdata-training/jetdata-mainpage.png){: width="500" height="400"}
_JETData.AI Main Page_

After you login into JETData.AI, you will see the main page. This page contains several button like:
1. Create New App
2. Create New Form
3. AI Forms Builder
4. Settings
5. User

and on the sidebar:
1. All Apps
2. Dashboard
3. Reports
4. Forms
5. Users
6. Log
7. Help

## Real World Application
Below are the 3 solution that I made using JETData.AI and n8n for the final of the training.

---

### Activity Report OCR

> The goal of this solution is to simplify the documentation of Activity Report for Engineer to Admin.

#### Demo Video

{% include embed/youtube.html id='q2KYydekpiA' %}

The very first steps to creating any form is creating new App. App in JetData is like a directory, where the inside of App is the form. **So in short App, group Forms together**

![Create New App](/assets/img/posts/jetdata-training/activityreport1.png){: width="250" height=""}
_Create New App Window_

After creating the App, we can continue with creating a new Forms, and we can choose the newly created App and create the form for that App.

![Create New Form](/assets/img/posts/jetdata-training/activityreport2.png){: width="250" height=""}
_Create New Form Window_

With the Form created inside the App, JETData will open a new windows to create a fields mapping. **What is fields mapping?**, fields mapping define how a field (Column) in one system or table to another field in other system.

![Create Fields Mapping](/assets/img/posts/jetdata-training/activityreport3.png){: width="250" height=""}
_Create Fields Mapping for Activity Report_

I've created the fields mapping for the acivity report OCR, after that you can start adding record inside the form with the ```Add Record``` button, and the add record will prompt you to insert the value you've put inside the fields mapping.

![Add Record Activity Report](/assets/img/posts/jetdata-training/activityreport4.png){: width="250" height=""}
_Add Records Button for Activity Report_

![Add Records Value Activity Report](/assets/img/posts/jetdata-training/activityreport5.png){: width="250" height=""}
_Add Records Value for Activity Report Item_

But, instead of manually adding records and entering the value for each item. This is where we will use ```Automation``` from JETData.AI that will do a HTTP POST request based on if the image is available or not.

![Add Automation for the Form](/assets/img/posts/jetdata-training/activityreport6.png){: width="250" height=""}
_Add Automation for Activity Report_

After pressing ```Add Automation``` a new window will appear, where you can configure how you want to automate your stuff, below are the automation used for Activity Report OCR where the condition will check whether if theres ```activityreport_image``` available or no in the fields, if the image is available then it will continue, if not then the automation does not run.

When the automation conditions are met, then the automation will do a HTTP Request POST to n8n webhook and also send the fields of ```id_record``` and ```activityreport_image``` to the webhook.

![Automation Configure](/assets/img/posts/jetdata-training/activityreport7.png){: width="250" height=""}
_Automation Configurations_

![Activity Report Automation Configuration](/assets/img/posts/jetdata-training/activityreport8.png){: width="250" height=""}
_Activity Report Automation Configuration_

With automation side done, we can move into the UI, with JETData.AI each form you can apply your own customized UI
by modifying the source with HTML, you can do this by right clicking the form ```Edit > Enable Customized Form > Modify Source```

> With Customized UI, it is Recommended to create a new FORM for UI
{: .prompt-tip}

![Edit Option on Form](/assets/img/posts/jetdata-training/activityreport9.png){: width="250" height=""}
_Form Edit Options_

![Customized UI Form](/assets/img/posts/jetdata-training/activityreport10.png){: width="250" height=""}
_Enabled Customized UI on FORM_

for the source code for Activity Report you can just copy paste this to have the exact thing as me.

> WARNING: NEVER PUT YOUR API KEY IN THE FRONTEND OR HARDCODE IT, THIS IS FOR TESTING PURPOSES ONLY!
{: .prompt-warning}

> Check Out The Source Code for Activity Report UI [HERE](/assets/source-code/jetdata-training/activityreportui.txt)
{: .prompt-info}

With customized UI enabled, now we're done with JETData.AI side, and can move on to the n8n side. Below are all the node used for the OCR, we'll be using ```webhook, AI Agent, OpenAI, and HTTP request tool```.

![Activity Report Workflow](/assets/img/posts/jetdata-training/activityreport11.png){: width="500" height=""}
_Activity Report Webhook_

For the ```Webhook```, we use POST for the binary image file to be processed by the AI Agent.

![Activity Report Webhook](/assets/img/posts/jetdata-training/activityreport12.png){: width="500" height=""}
_Activity Report Webhook_

For the ```AI Agent```, on the prompt (User Message), we can use the actual image and the record id for the row inside the JETData.

![Activity Report AI Agent](/assets/img/posts/jetdata-training/activityreport13.png){: width="500" height=""}
_Activity Report AI Agent_
This is for User Prompt:
{%raw%}
```md
use the {{ $json.query.activityreport_image }} and do the ocr.

after that use the tool searchUpdateTool and use the id_record to know which record to update {{ $json.query.id_record }}
```
{%endraw%}

This is for System Message:
{%raw%}
```md
You are an office administrative AI assistant responsible for extracting structured data from Activity Report documents (PDF or image) using OCR.

Your task is to extract values only if they are explicitly visible, clearly readable, and unambiguous in the document.

STRICT RULES (NO EXCEPTIONS)

DO NOT hallucinate, guess, infer, normalize, or auto-complete any value.

If a field is missing, unclear, unreadable, partially visible, or doubtful, set the value to null.

Use only the exact field names defined below.

Do NOT add, remove, rename, or merge fields.

Return valid JSON only — no explanations, no markdown, no comments.

Extract text exactly as written. Do not rephrase.

FIELD DEFINITIONS (USE EXACT KEYS)

{
"activityreport_image": null,
"customer_name": null,
"customer_address": null,
"activity_type": null,
"start_datetime": null,
"finish_datetime": null,
"product_type": null,
"product_name": null,
"serial_no": null,
"serial_tag": null,
"description_ar": null,
"symptom_ar": null,
"action_ar": null,
"customer_signature_name": null,
"engineer_name": null,
"approval": null
}

FIELD-SPECIFIC RULES

activityreport_image
Do not describe the image.
Only set a value if a clear image reference or filename is explicitly provided.
Otherwise, use null.

activity_type
Only extract if the value is explicitly written (for example: Installation, Maintenance).
Do not guess based on context.

start_datetime and finish_datetime
Extract only if both date and time are clearly written.
Do not convert formats.
If either date or time is missing or unclear, use null.

description_ar, symptom_ar, action_ar
Extract only if a clearly written section exists.
If handwriting or OCR text is uncertain, use null.

approval
Set only if the document explicitly states an approval status (for example: Approved, Rejected).
Signatures alone do NOT imply approval. if theres none, set default to Pending

created_date
Never extract this field.
Always ignore it.

EXTRACTION PHILOSOPHY

Accuracy is more important than completeness.
Null is always preferred over an incorrect value.
The output will be sent directly to an API without human review.

Output ONLY the JSON object.

run the tool searchUpdateTool
```
{%endraw%}

When the OCR is done, then the AI Agent will use the ```searchUpdateTool``` to update the row with the value generated from the OCR.

![Activity Report HTTP Request Tool](/assets/img/posts/jetdata-training/activityreport14.png){: width="500" height=""}
_Activity Report HTTP Request Tool_

And that's it!, below are the screenshot for the UI

![Upload Activity Report](/assets/img/posts/jetdata-training/activityreport15.png){: width="500" height=""}
_Upload Activity Report_

![View All Record Activity Report](/assets/img/posts/jetdata-training/activityreport16.png){: width="500" height=""}
_View All Record Activity Report_

---

### Overtime Eligibility Checker and Form Generator

> The goal of this solution is to help Engineer with creating the printed form and checking the eligibility of the overtime being submitted.

#### Demo Video

{% include embed/youtube.html id='RGuQQEuM79o' %}

Again, with every new app, we create a form for it!, one for backend form, and one for frontend form. For backend form I'll be naming it ```Bucket - Overtime``` and frontend ```UI - Overtime```.

![Overtime Form](/assets/img/posts/jetdata-training/overtime1.png){: width="250" height=""}
_Overtime Form_

With the form created, in the backend form, we add fields mapping that will be used for checking the eligibility and if its approved or no.

![Overtime Form Fields Mapping](/assets/img/posts/jetdata-training/overtime2.png){: width="500" height=""}
_Overtime Form Fields Mapping_

Fields mapping are done, now we can customize the frontend form, if you want the same looking UI, you can use the same source code HTML as shown

> WARNING: NEVER PUT YOUR API KEY IN THE FRONTEND OR HARDCODE IT, THIS IS FOR TESTING PURPOSES ONLY!
{: .prompt-warning}

> Check Out The Source Code for Overtime UI [HERE](/assets/source-code/jetdata-training/overtimeui.txt)
{: .prompt-info}

after frontend, we want to add an automation on JETData.AI where it will do HTTP request POST to n8n.

![Overtime Form Automation](/assets/img/posts/jetdata-training/overtime3.png){: width="500" height=""}
_Overtime Form Automation_

With the JETData side done, we move on to n8n. Inside n8n will be used to check the eligibility of the overtime that are submitted, and update the approval record.

![Overtime n8n Workflow Overview](/assets/img/posts/jetdata-training/overtime4.png){: width="500" height=""}
_Overtime n8n Workflow_

The n8n ```webhook``` will capture the sended HTTP Request from the automation, and will process it in the workflow.

![Overtime n8n Webhook](/assets/img/posts/jetdata-training/overtime5.png){: width="500" height=""}
_Overtime n8n Webhook_

Inside the ```AI agent```, we will give the AI agent a prompt that will use the specified value:

{%raw%}
```md
id_record: {{ $json.body.id_record }}
customer_name: {{ $json.body.customer_name }}
start_datetime: {{ $json.body.start_datetime }}
finish_datetime: {{ $json.body.finish_datetime }}
engineer_name: {{ $json.body.engineer_name }}
activity_type: {{ $json.body.activity_type }}
description_ar: {{ $json.body.description_ar }}
```
{%endraw%}

for the system message we will use this:

{%raw%}
```md
You are a helpful assistant

First check the eligibility of the activity report with Overtime Eligibility Check AI agent to check whether its inside the time frame of the eligible overtime claim.
```
{%endraw%}

![Overtime n8n AI Agent](/assets/img/posts/jetdata-training/overtime6.png){: width="500" height=""}
_Overtime n8n AI Agent_

Then we will use the AI Agent tool to check if overtime is eligible or not,  the AI agent use the ```Date & Time``` tool to be able to check the time whether it is outside work date/time or still inside the time frame.

![Overtime n8n AI Agent Tool](/assets/img/posts/jetdata-training/overtime7.png){: width="500" height=""}
_Overtime n8n AI Agent Tool_

Last step is updating the record whether it is approved, or rejected, by using the ```searchUpdate``` tool.

![searchUpdate Overtime Record](/assets/img/posts/jetdata-training/overtime8.png){: width="500" height=""}
_searchUpdate Overtime Record_

Finally we're done!, check below for screenshot of the working UI & Backend.

![Adding Record for Overtime Testing](/assets/img/posts/jetdata-training/overtime9.png){: width="500" height=""}
_Adding Record for Overtime Testing_

![Overtime UI](/assets/img/posts/jetdata-training/overtime10.png){: width="500" height=""}
_Overtime UI_

![Overtime UI Generated Form](/assets/img/posts/jetdata-training/overtime11.png){: width="500" height=""}
_Overtime UI Generated Form_


---

### Support Ticket Chatbot

> The goal of this solution is to help Engineer with creating a Support Ticket with limited subject or description.

#### Demo Video

{% include embed/youtube.html id='6R0GVhgma1k' %}

First steps, of course, Forms!, we need to create the Form for the backend where it will hold the data, and the Form for Customized UI. For backend form I'll be naming it ```Bucket - Support``` and UI ```UI - Support```

![Support Backend Form](/assets/img/posts/jetdata-training/supportticket1.png){: width="250" height=""}
_Creating Backend Support Form_

![Support Frontend](/assets/img/posts/jetdata-training/supportticket2.png){: width="250" height=""}
_Creating Frontend Support Form_

After Forms is created, with the backend form, we can add the fields for the data that we want, below are the fields that I will use.

![Support Ticket Fields Mapping](/assets/img/posts/jetdata-training/supportticket3.png){: width="500" height=""}
_Backend Support Form Fields Mapping_

Now we can continue with the frontend Form, again we can customize the Form by right clicking the form ```Edit > Enable Customized Form > Modify Source```. inside the "Modify Source" you can use the exact code as mine below.

> WARNING: NEVER PUT YOUR API KEY IN THE FRONTEND OR HARDCODE IT, THIS IS FOR TESTING PURPOSES ONLY!
{: .prompt-warning}

> Check Out The Source Code for Support Ticket UI [HERE](/assets/source-code/jetdata-training/supportui.txt)
{: .prompt-info}

With all the JETData side completed, we can continue to the n8n side. n8n will be used for the workflow, generating responds, and managing the responded data back to JETData.

![Support Ticket n8n Workflow](/assets/img/posts/jetdata-training/supportticket4.png){: width="500" height=""}
_Support Ticket n8n Workflow_

![Support Ticket Chat Trigger](/assets/img/posts/jetdata-training/supportticket5.png){: width="500" height=""}
_Support Ticket Chat Trigger_

For the AI Agent, I use two tools, addRecords and searchRecord. for the AI Agent, on Prompt User Message you can insert this:

{%raw%}
```md
userName: {{ $json.userName }}
chatInput: {{ $json.chatInput }}

when asked who am i, use userName

for the engineer_name use userName

you can guess and elaborate a bit more for the best practice for the issues and demands, action takens, and conclusion
```
{%endraw%}

This prompt will make the AI Agent use the ```userName``` that we will get from the chat in JETData, and the ```chatInput``` to be used for the 

![Support Ticket AI Agent](/assets/img/posts/jetdata-training/supportticket6.png){: width="500" height=""}
_Support Ticket AI Agent_

With the tool ```addRecord```, we will use POST method, and configure the query and body parameter. For the body parameter, just use the same name as what we have on fields mapping, and for the value just use ```Define automatically by the model```

![addRecord Query Parameter](/assets/img/posts/jetdata-training/supportticket7.png){: width="500" height=""}
_addRecord Tool Query Parameter_

![addRecord Body Parameter](/assets/img/posts/jetdata-training/supportticket8.png){: width="500" height=""}
_addRecord Tool Body Parameter_

For the tool ```searchRecord```, we can use the API from the JETData to search for the record.

![searchRecord Tool](/assets/img/posts/jetdata-training/supportticket9.png){: width="500" height=""}
_searchRecord Tool Body Parameter_

And We're Done!, now we can test the chatbot with creating a support ticket, with a simple description of the issues, and check whether the generated support ticket is logged into the backend form.

![Using Support Ticket Chatbot](/assets/img/posts/jetdata-training/supportticket10.png){: width="500" height=""}
_Using Support Ticket Chatbot_

![Generated Support Ticket](/assets/img/posts/jetdata-training/supportticket11.png){: width="500" height=""}
_Generated Support Ticket_

![Logged Generated Support Ticket](/assets/img/posts/jetdata-training/supportticket12.png){: width="500" height=""}
_Logged Generated Support Ticket_

## Conclusion

I hope you find this write-up or guide helpful in some way. Thank you!