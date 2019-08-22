---
title: Enabling Pushover Notifications on Successful SSH Logins
author: Jonah
layout: post
canonical: "https://write.privacytools.io/jonah/enabling-pushover-notifications-on-successful-ssh-logins"
tags:
  - security
  - sysadmin
  - guides
---

If you run servers for public services, like I do with privacytools.io, you definitely want to monitor them for any successful logins to user accounts (via SSH, et cetera). The way I plan to accomplish this is to setup an automatic notification to the Pushover app on my phone in the event of any login. I'm going to implement this as part of PAM authentication, and configure it to fail any logins if the notification script fails for whatever reason.<!--more-->

If you want to implement this on your own server, I'll assume you already have a Pushover account and device setup, and an API key.

Create `login-notify.sh`, where we will store the actual script. I put it in `/etc/ssh/` for example but you could put it anywhere:

```
#!/bin/bash

# Change these variables
API_TOKEN=abcdefg1234hijklmno567890pqrstuv
API_USER=vutsrqp098765onmlkjih4321gfedcba

if [ "$PAM_TYPE" != "close_session" ]; then
  TITLE="SSH: ${PAM_USER}@$(hostname -f) (${PAM_RHOST})"
  TEXT="$(date)"

  curl -s \
  -F "token=$API_TOKEN" \
  -F "user=$API_USER" \
  -F "title=$TITLE" \
  -F "message=$TEXT" \
  -F "priority=0" \
  https://api.pushover.net/1/messages.json >/dev/null 2>&1
fi
```

Just change `API_TOKEN` and `API_USER` to your own account's values. You can also change `priority=0` to [another value](https://pushover.net/api#priority) if you'd prefer more or less intrusive notifications.

Make your script executable:

```
chmod +x login-notify.sh
```

And add the following line to the end of `/etc/pam.d/sshd`:

```
session optional pam_exec.so seteuid /path/to/login-notify.sh
```

We made this `optional` mainly for testing purposes. You can leave it as it is, or change it to `required` *after you've made sure it works* to prevent logins entirely unless the script runs, if that is what you want.

Try logging in to SSH and it should send you a notification!

In theory, this method can also be applied to essentially any `/etc/pam.d/` module. For example, you could add that last line to `/etc/pam.d/login` for notifications on TTY logins.

Thanks to [this answer from Fritz on Ask Ubuntu](https://askubuntu.com/a/448602) and [this post on Nology](https://nology.de/ssh-automatic-notifications-login-pushover.html) for guidance with the script.
