---
title: My e-mail setup
date: 2021-03-14
tags:
  - software
  - linux
  - email
  - english
layout: layouts/post.njk
---

## Background

I created my first email address in 1996 or 1997. It was a free account from traveltales.com, a service that does not exist anymore. At the beginning, email was so impressive. You could communicate with people all around the world. In fact, on 1998, I remember to receive emails from a family's friend who was studying in Germany at the time. We felt like living in the future, a feeling I still have but would be funny for the tech native generation. For them, of course, internet is as normal as having socks.

Later, I got an email address from our internet provider. It was mjonline@entelchile.net. **M** stands for Mauricio and **J** stands for Juan, since we had only one address per account, I shared this address with my dad, Mauricio. This email address did not include any webmail interface, as long as I can recall, so we used [Netscape Mail & Newsgroups](https://en.wikipedia.org/wiki/Netscape_Mail_%26_Newsgroups) for Windows. It was so slow in our old 486 Cyrix computer that we usually wrote the email messages offline in Notepad and the pasted it to the Netscape composer.

I started using Linux on 1998, mainly because I was curious and had a lot of free time. On Linux, Netscape worked even better but remember to have used also [pine](https://en.wikipedia.org/wiki/Pine_(email_client)) and other email clients. In the meantime, I had multiple email addresses from free providers like Yahoo!.

Everything changed in 2004 or 2005 when my sister and I got our GMail accounts. My sister was lucky to get her *first name* username. I got *juancri* which has been my nickname since I was a kid.

The GMail web interface was incredible and, with 1GB of storage, which seemed infinite at the time, we didn't feel any need to delete old emails. Also, thanks to its revolutionary search feature, having a ton of incoming emails wasn't an issue so both factors -the storage and search feature- contributed to the bad practice of signing up to receive a lot of crap emails.

Now, in 2021, being older and trying to optimize my workflows, I decided to follow another path. I'm leaving GMail to use [neomutt](https://neomutt.org/), which is a *command line mail reader (or MUA). It’s a fork of Mutt with added features*.

Now, there are multiple reasons. I can't deny that one of them is that I think it's fun. Another one has to do with quit depending on GMail. I usually use my GMail email address as a username for websites and forward email messages to my own domain. I want to stop that practice and just using my own email address.

This raises a question. Am I willing to use time to admin an email server. NO. I decided to use a cloud anyway. Here are the details.

## Email server

I don't want to be the admin of any servers. Life is too short for that. That's why I host this blog using AWS S3 and CloudFront (I should have posted about this, but I probably didn't, so I'll leave this note as a reminder).

After analyzing multiple options, I decided to try with [AWS WorkMail](https://aws.amazon.com/workmail/), which is Amazon's hosted email service. It provides everything I need: SMTP and IMAP.

So, just in case you're not familiar with email protocols, you usually need to configure two *servers* for your email client. One server to send email and another one to receive email. For sending email, there's only one popular protocol: SMTP (simple mail transfer protocol). To receive email, there are two main protocols: POP and IMAP. POP is a protocol from which you just download your email, store it locally and the remote copy is usually deleted. IMAP is different, since you "sync" your local folders with the IMAP server. This is more similar to GMail and other managed services, allowing multiple clients to receive emails and been synced to the server.

AWS WorkMail offers both protocols, as well as an optional web interface that I will not be used for now. The [pricing structure](https://aws.amazon.com/workmail/pricing/) is fairly simple: *$4.00 per user per month and includes 50 GB of mailbox storage for each user*. This is OK to me. My AWS monthly bill is usually less than a dollar, so I'm OK paying for email hosting. As mentioned before, hosting is not a big issue. You can get a VPS for USD 5 a month. The real issue is administrative tasks. So, I'm happy to pay 4 bucks so AWS staff manages the servers.

Having my own domain name also gives me freedom to move to another provider at any time. I migrated from GMail to AWS WorkMail in a couple of hours so it's generally a pretty simple process.

## Email client

Since AWS WorkMail (too long, so I'll use AWS as a synonym from now forward) supports IMAP and SMTP, standard protocols (BTW, both defined by IETF RFCs: [SMTP](https://tools.ietf.org/html/rfc5321) / [IMAP](https://tools.ietf.org/html/rfc3501)), I'm free to use any email client.

Since I think I'm cool and hipster, and as mentioned before, I'm trying to use neomutt. First I configured it manually, but then tried this [mutt wizard](https://github.com/LukeSmithxyz/mutt-wizard) created by Luke Smith (BTW, check [his awesome YouTube channel](https://www.youtube.com/channel/UC2eYFnH61tmytImy1mTYvhA)).

This wizard configures a lot of things but I had to fix a few issues with the ports and protocols for AWS.

This is the full command line:

```
mw -a juancri@juancri.com \
  -n "JC Olivares" \
  -i imap.mail.us-west-2.awsapps.com \
  -I 993 \
  -s smtp.mail.us-west-2.awsapps.com \
  -S 465 \
  -o
```

After that, I just had to add the option `tls on` to the msmtp config file. As a reference, this is my current msmtp config file (`~/.config/msmtp/config`):

```

account juancri@juancri.com
host smtp.mail.us-west-2.awsapps.com
port 465
from juancri@juancri.com
user juancri@juancri.com
passwordeval "pass juancri@juancri.com"
auth on
tls_starttls off
tls on
tls_trust_file	/etc/ssl/certs/ca-certificates.crt
logfile /home/juancri/.config/msmtp/msmtp.log
```

Be aware that the specific host will depend on the region where your domain was configured on AWS. Check [this page](https://docs.aws.amazon.com/workmail/latest/userguide/using_IMAP.html) for details.

Something else that is not mentioned in the README is how to setup the cron to sync email. This is my current crontab:

```
# mm  hh  DD  MM  W /path/progam [--option]...  ( W = weekday: 0-6 [Sun=0] )

  */5 *   *   *   * mw -Y
```

That's it! I also configured [thunderbird](https://www.thunderbird.net) which I might be using from time to time.
