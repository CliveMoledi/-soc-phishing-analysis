# PHISHING ANALYSIS

## Objectives

- Analyze the provided email samples to identify key artifacts
- Investigate phishing URLs to understand redirection
- Retrieve and examine the phishing kit used in the attack
- Use CTI tools to gather intelligence on the adversary
- Analyze the phishing kit to uncover additional indicators

<img width="2880" height="1348" alt="Screenshot 2026-06-15 at 11 39 22" src="https://github.com/user-attachments/assets/cd87e6ad-3e98-4e03-bf48-31f033b84bc4" />
I have a set of 5 emails. I'm gonna start by investigating "Quote for Services Rendered" email.


<img width="2879" height="1344" alt="Screenshot 2026-06-15 at 11 41 53" src="https://github.com/user-attachments/assets/04a80509-0009-4305-9577-5cb8405fe5a9" />
Let's open the email.We have a plathera of information we can work with. The most fundamenatal artifact to lookout for is who the email was sent to, because the recipient list tells you the scope and intent of the attack. 

- The recipient is: William McClean.
- We can also see that the email of the sender is : Accounts.Payable@groupmarketingonline.icu

---
We have an email that was sent to Zoe Duncan that has a html attachment which i'm interested in. 
<img width="2880" height="1530" alt="Screenshot 2026-06-15 at 13 09 37" src="https://github.com/user-attachments/assets/a4b8b2ae-60a0-461b-ad98-014a85246fbb" />
I went ahead and downloaded the file and opened it.NB this is a sandboxed envirnment that's why i opened the file.
<img width="2880" height="1353" alt="Screenshot 2026-06-15 at 12 23 11" src="https://github.com/user-attachments/assets/a2aa4971-7a0f-4a6c-b101-c1dc3a28db75" />

The web page looks like a regular microsoft login page for the organisation. One thing that stoodout was the root domain. It was not from Microsoft and that screamed redflag. I need to further investigate this.
<img width="2880" height="1566" alt="Screenshot 2026-06-15 at 12 59 09" src="https://github.com/user-attachments/assets/3f09a25b-fe00-4cbc-8535-4e2c82b2ad7c" />

---

I Navigated to the /data directory to see if i can find something of interest<img width="2880" height="1537" alt="Screenshot 2026-06-15 at 13 30 44" src="https://github.com/user-attachments/assets/f70636dc-33b1-41cc-b94d-4447132aac0e" />
.
I went i ahead and downloaded this file(Their phishing kit)

<img width="1522" height="1528" alt="Screenshot 2026-06-15 at 13 35 52" src="https://github.com/user-attachments/assets/fc4470a3-dedc-42da-b050-756e60a84ebb" />

went to terminal and got the sha256 hash: ba3c15267393419eb08c7b2652b8b6b39b406ef300ae8a18fee4d16b19ac9686
<img width="626" height="480" alt="Screenshot 2026-06-15 at 13 55 10" src="https://github.com/user-attachments/assets/9b041cd1-9731-4935-adc1-b7d092587249" />

I'm gonna go ahead and paste it on [Virus Total](https://www.virustotal.com/gui/home/search) to quickly determine whether a file is known and malicious.
<img width="709" height="661" alt="Screenshot 2026-06-15 at 14 01 48" src="https://github.com/user-attachments/assets/3bbb9a9e-0750-4f4e-a83e-d1b69f9c502d" />

Update365.zip is a confirmed malicious true positive, with 30 out of 62 vendors classifying it as a Trojan and a phishing delivery mechanism.The archive uses masquerading as an Office 365 update to deceive targets, making it a high-risk indicator for credential harvesting.

<img width="2880" height="1434" alt="Screenshot 2026-06-15 at 14 04 19" src="https://github.com/user-attachments/assets/5f08f570-6fcf-439d-af4a-69842b63d6f1" />

---

I want to go ahead and open the phishing kit(/Update365/office365/Validation/) 
to look around and discover what email address is used by the adversary to collect compromised credentials.Which I found the Update365/office365/Validation/submit.php is the one responsible for submit button which will send email to m3npat@yandex.com

<img width="1435" height="688" alt="Screenshot 2026-06-15 at 14 45 48" src="https://github.com/user-attachments/assets/e2258035-65df-4fca-a3bc-823b22abd3ee" />




