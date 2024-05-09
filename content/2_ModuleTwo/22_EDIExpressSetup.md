---
title: "Express for EDI Setup" 
chapter: true
weight: 1 
---

# Express for EDI Setup


##  Express for EDI on AWS Marketplace

1. Navigate to [Express for Energy Data Insights is available on AWS Marketplace](https://aws.amazon.com/marketplace/pp/prodview-n3hoeanhhzcmm)

2. Click on "View purchase options"
![Express for EDI](/images/ExpressForEDI_Marketplace.PNG)

3. Click on "Subscribe"
![Express for EDI - Purchase options](/images/edi_subscribe_1.png)

4. Click on "Setup your account"
![Express for EDI - Subscribe](/images/edi_subscribe_2.png)

5. If using Express for EDI for the first time, Create Account. Otherwise, Sign In.
![Express for EDI - Sign in / Subscribe](/images/edi_subscribe_5.png)

6. After logged in, fill out the necessary information needed for your subscription. It is recommended to select "Multi-tenant" for Tenancy option to save costs and speed up environment provisioning for the current workshop. Once all parameters are filled out, click "Next".
![Express for EDI - Sign in / Subscribe](/images/edi_subscribe_3.png)

7. The environment will be provisioned and you should be notified by e-mail. 

8. After your EDI instance is provisioned please login to your EDI instance with the instructions provided via e-mail. 

9. In order to connect your application, you need to be able to Authenticate to your EDI instance. You will need AWS Cognito information and your EDI paritition Id for this. 

10. Using the Support widget at bottom right of EDI portal screen, please open a Support ticket.
![Express for EDI - Support Ticket](/images/edi_subscribe_6.png)


11. In the support request, please ask for App Client ID, Client Secret, EDI Instance URL, EDI Paritiion Id and AWS Region where EDI Instance is running to be provided.   
![Express for EDI - Support Ticket](/images/edi_support_ticket.jpg)

12. Once, you receive all the required information from Support, you are ready to proceed with the workshop.

### EDI Core APIs
Next, we will use the Express for EDI instance which we have signed up for to explore the EDI APIs.

