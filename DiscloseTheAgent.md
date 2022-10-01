# Disclose The Agent

Write up for the "Disclose The Agent" challenge on letsdefend.io

Challenge can be [found here](https://app.letsdefend.io/challenge/disclose-the-agent/)
if you want to have a go yourself.

Before starting this challenge there is a [prerequisite pcap file (pass = 321)](https://app.letsdefend.io/download/downloadfile/smtpchallenge.zip) that must be downloaded. 

This file only contains web packet logs so it is safe to download onto your main machine, unlike some other challenges on letsdefend, which contain live malware and should only be downloaded onto an isolated enviroment.

Open up the pcap file in any suitable software you wish, I will personally use WireShark, and we can begin with the first question.

## Question 1
```
What is the email address of Ann's secret boyfriend?
```
When dealing with emails in relation to web packet analysis, we should instantly think SMTP. SMTP stands for Simple Mail Transfer Protcol and is the Internet standard protocol for email sending and receiving. We can therefore safely scroll through the logs and begin inspecting the SMTP packets more thoroughly.

<img src="https://user-images.githubusercontent.com/106356256/193426660-db0d0297-a654-450c-bcb9-a485b25c23ca.png" style="height: auto; width:auto;"/>

Luckily Ann has only sent one email, so we don't have to do any detective work to find out who her secret boyfriend is.

## Question 2
```
What is Ann's email password?
```
<img src="https://user-images.githubusercontent.com/106356256/193426675-47f9af48-34a4-42e8-bfec-f391b6569e4e.png" style="height: auto; width:auto;"/>


We first see an EHLO packet from annlaptop. It is safe to assume this is the ann that is spoken of in the question. We also see an authentication attempt with a username and password being sent. 
<img src="https://user-images.githubusercontent.com/106356256/193426692-832f6be8-d999-4ca1-8462-2f7d0f0eb0ef.png" style="height: auto; width:auto;"/>

Upon doing some research we find out that the default auth command leaves these values in plain text with base64 encoding, this means we can easily decode them, thus giving us the answer to the question (if this is actually Ann's authentication attempt). 

Upon decoding we get the following values 
```sneakyg33k@aol.com``` and ```558r00lz```

The email matches with the email sent to her secret boyfriend, so we can assume that ```558r00lz``` is her password.

## Question 3
```
What is the name of the file that Ann sent to his secret lover?
```
<img src="https://user-images.githubusercontent.com/106356256/193426700-50084bd0-0d69-446a-8817-cf3360da6bfe.png" style="height: auto; width:auto;"/>

To see any attachements that ann sent, we must take a deeper dive into the compiled SMTP packet of the rendezvous email she sent. 

<img src="https://user-images.githubusercontent.com/106356256/193426706-4b235296-d6ef-4cae-b289-d741dcd28614.png" style="height: auto; width:auto;"/>

As you can see, she attached a file called secretrendezvous.docx to her email.

## Question 4
```
In what country will Ann meet with her secret lover?
```

From the previous question, we read Ann's email and now know that her meetup location is in the secretrendezvous.docx file. Luckily, we have captured all of the TCP data coming from her pc and as such we can simply reconstruct the file. We can use a programe called ```Network Miner``` to do this.

<img src="https://user-images.githubusercontent.com/106356256/193426741-db16fa1f-ba75-4826-bd7b-39243f78eccf.png" style="height: auto; width:auto;"/>

Network miner has successfully reconstructed the file (as well as others) from the pcap file we have been using. Upon opening the file we see her meet up location.

<img src="https://user-images.githubusercontent.com/106356256/193426758-7d47c86d-821a-4fb5-8954-d573423dd011.png" style="height: auto; width:auto;"/>


## Question 5
```
What is the MD5 value of the attachment Ann sent?

```
To get the MD5 value, we can simply use CMD on the reconstructed file.

<img src="https://user-images.githubusercontent.com/106356256/193426766-6ee6f83a-60eb-4eed-b45e-e66b679d0d4d.png" style="height: auto; width:auto;"/>
