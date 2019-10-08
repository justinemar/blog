---
title: setup free professional email domain for your web apps
date: 2019-10-05 21:52:39
tags:
---

# Prerequisite
This writing targets people who already have experience with creating NodeJS Apps since i won't go on detail explaining things.


# Intro
So you have a hobby project? and you want a custom email domain tailored to your app niche? Good news! you don't need to buy a web hosting or a domain!, With `mail.com` which offers a lot of custom email domains for free you can easily create a professional looking email for your projects. But where is this good for? Well that really depends on you, but in my scenario i wanted to send email confirmation to users who registered to my app, using gmail alone wasn't good enough for me because i wanted the email to look professional. So what i did is hooked up my `mail.com` account to my gmail account.

Heres what it looked like.
![email confirmation](/images/verification.PNG)

# Setup

 1. You can setup gmail to send your emails using your mail.com account.
 #### Advantages
 - No need to handle more than one email
 - You can securely authenticate with OAuth or with App passwords


 2. You can directly use your mail.com registered account (using your actual login credential)

 Whatever you choose you still need to do this: go and register first to [mail.com](https://mail.com) and choose the email domain best suited for your need. 


### Linking mail.com to your gmail.

Open [your gmail](https://mail.google.com) and click the gear icon at top right corner then click on settings and choose the `Accounts and Imports` tab, Now for the `Send email as` click add another email address and this should open a new window simply input your desired name and the email address you registered at mail.com and make sure you checked the 'Treat as an alias' Just click next, enter your mail.com registered account detail and set the smtp server as `smtp.mail.com` with port 587.

![setup format](/images/bida.PNG)

After you confirm, google will send you a verification code to your mail.com account so go and get the code there. Once you verified yourself go back to gmail setting for the `Send email as` make your mail.com account as a dfault email.

![make it default](/images/default.PNG)

Now you should be able to send emails using your mail.com account directly from gmail! 

### Confirming users email

Let's create a simple REST API that creates a user and sends an email confirmation.

Require these packages
```
const nodemailer = require('nodemailer'),
    http = require('http'),
    fs = require('fs'),
    { parse } = require('querystring'),
```

We'll use a mock data to simulate our database
```
const users = [{ email: 'aba@gmail.com', pass: 'b', verified: true }, { email: 'kada@gmail.com', pass: 'c', verified: false }];
```
In the code below Add your gmail login credentials in the `auth` property. It is best to store your login credentials to a safe place such as env variable or if you want a higher security you can use [Oauth2](https://nodemailer.com/smtp/oauth2/) or create an[app password](https://support.google.com/accounts/answer/185833?hl=en)

In the real world you send a temporary verification token which belongs to the account that needs to be verified, if you are using JWT you can have a signed token which holds a unique identifier (email or id) for the account and validate the token in the server. But this time we'll only verify the user if it is a valid email in the mock database to make things simple.

```
verifyUser = (email) => {
    const user = users.find(user => user.email === email);
    if (user && !user.verified) {
        user.verified = true;
    }
}
sendConfirmation = (email) => {
    const transporter = nodemailer.createTransport({
        service: 'gmail',
        host: 'smtp.gmail.com',
        auth: {
            user: 'yourmail@gmail.com',
            pass: 'aha'
        }
    });

    const mailOptions = {
        to: email,
        from: 'bida@asia.com',
        subject: 'Account Verificaiton',
        text: 'Hey you want to secure your account? too bad i aint got time for that.'
    };

    transporter.sendMail(mailOptions, (error, info) => {
        if (error) {
            console.log(`Could not send mail ${error}`);
        }

        if (info) {
            res.json({
                message: 'An email containing your verification link has been sent.',
                type: 'success',
                code: 200,
            });
        }
    });
}


http.createServer(function (request, response) {

    let body = '';
    if (request.url.indexOf('?') >= 0) {
        const query = parse(request.url.replace(/^.*\?/, ''));
        console.log(query.email)
        const user = users.find(user => user.email === query.email);
        if (user) {
            verifyUser(query.email)
        }
        response.end()
    }

    if (request.url === '/api/account' && request.method === 'POST') {
        response.setHeader("Content-Type", "text/html");

        request.on('data', chunk => {
            body += chunk.toString();

        })

        request.on('end', _ => {
            const { password, email } = parse(body);
            users.push({ email, password, verified: false });
            sendConfirmation(email)
            response.end('Email confirmation sent.')
        })

    }

    if (request.url === '/api/login' && request.method === 'POST') {
        response.setHeader("Content-Type", "text/html");
        request.on('data', chunk => {
            body += chunk.toString();

        })

        request.on('end', _ => {
            const { email } = parse(body);
            const user = users.find(user => user.email === email);

            if (!user) {
                response.end('No user')
            }

            if (user && user.verified) {
                response.end('nice you are verified')
            } else {
                response.end('Not verified')
            }


        })

    }
}).listen(8000);
```

The rest of the code is pretty much self explanatory, but if everything works well this is how it looks

![email](/images/email2.PNG)
![email](/images/email.PNG)

You can use an email template to make it pretty :)

![email confirmation](/images/verification.PNG)