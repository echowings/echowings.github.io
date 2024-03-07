---
title: "How to Send Email With Sendgrid Api"
date: 2024-03-07T14:47:50+08:00
draft: false
image: https://picsum.photos/800/600.webp?random=240a31f5
categories:
  - python
tags:
  - python
  - api
  - sendgrid
layout: 
  - post
slug: 
  - post

license: CC BY-NC-ND
---



```python
import sendgrid
#import os
from sendgrid.helpers.mail import Mail, Email, To, Content

#export SENDGRID_API_KEY="API_KEY"
#echo $SENDGRID_API_KEY
SENDGRID_API_KEY="API_KEY"
print(SENDGRID_API_KEY)
#sg = sendgrid.SendGridAPIClient(api_key=os.environ.get(SENDGRID_API_KEY))
sg = sendgrid.SendGridAPIClient(api_key=SENDGRID_API_KEY)
from_email = Email("sender@sender.com")  # Change to your verified sender
to_email = To("receiver@receiver.com")  # Change to your recipient
subject = "Sending with SendGrid is Fun"
content = Content("text/plain", "and easy to do anywhere, even with Python")
mail = Mail(from_email, to_email, subject, content)

# Get a JSON-ready representation of the Mail object
mail_json = mail.get()

# Send an HTTP POST request to /mail/send
response = sg.client.mail.send.post(request_body=mail_json)
print(response.status_code)
print(response.headers)
```