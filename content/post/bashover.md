---
title: "Pushover BASH script - Bashover"
date: 2014-12-10T00:00:00+01:00
draft: false
cover: "/images/bo-sh/title.png"
---

![Using Bashover in a Linux shell](/images/bo-sh/bosh01.png)

Recently, I had an issue with my web server, the one hosting this site, where MySQL died. It wasn’t until several (16) hours later, when my Android Email app notified my of failed logins, that I realised there was a problem.


For the entirety of this time, anything hosted on this box had been without a database connection, which obviously meant this blog was also down. The first thing that struck me, when wading through the logs to find out what went wrong, was that I had known absolutely nothing about the problem for so long.

It was immediately apparent to me, that what I required was a form of push message to be sent to my phone. I didn’t want to write entire application for push capability, and I don’t want to send emails either. Having been aware of Pushover, and being a fan due to its one off charge for the client application, I decided I needed to use it.

In order for me to send messages about what’s going on, I’d need a script, separate from  the important logic which determines what’s working and what isn’t that could take a message and simply push it to my phone.

# The Script

``` sh
#!/bin/bash

LOGGING=true # This will print the message and response to the syslog.
URL="https://api.pushover.net/1/messages.json"

APP_KEY="" #token
USER_KEY="" #user
TITLE="Alert"
PRIORITY=1
MESSAGE=$*

if [ ${#MESSAGE} == 0 ]
then
MESSAGE=$(cat)
fi

if [ ${#APP_KEY} == 0 ]
then
echo "Error: You need to supply an app token"
exit 1;
fi

if [ ${#USER_KEY} == 0 ]
then
echo "Error: You need to supply a user key"
exit 1;
fi

RESPONSE=`curl -s --data token=$APP_KEY \
        --data user=$USER_KEY \
        --data-urlencode title="$TITLE" \
        --data priority=$PRIORITY \
        --data-urlencode message="$MESSAGE" $URL`
        
if [ $LOGGING == true ]
then
logger "PUSH"
logger "|"
logger "| MESSAGE: $MESSAGE"
logger "| RESPONSE: $RESPONSE"
logger "|______________________________________________________________"
fi

exit 0;
```

As you can see, this is a very simple script. It’s a barebones script, and the idea is that it can be run on most systems without the need of additional tools. You will of course need the curl package, which is available on most if not all linux distros.

A bunch of Pushover features are missing, I don’t have any real intention to add them to this particular script, as they are unnecessary, this is intentionally kept simple, to be used along with a cron job, but feel free to edit it to your requirements.

There is a little bit of code for some basic error handling, in case you forget to set the keys, but otherwise very little. Since I didn’t want to require a third party tool to parse the JSON responses from the API, they’re simply logged, untouched, to syslog (if enabled).

I’ll perhaps create an advanced version of this script that parses the response and acts accordingly, which will probably require something like jq.

# Usage
Once configured, there are 3 basic ways to use this script;

Call it with a message:

``` sh
./bo.sh "Hello, 2bytes"
```

![Notification received on mobile phone](/images/bo-sh/bosh02.png)

Call it without a message:

``` sh
./bo.sh
```

Doing so will leave you at an entry prompt, this can be useful if you want to manually send something with formatting. You can enter text, and hit enter, creating new lines, and it will continue to take input until you press CTRL+D. It will then push the entire message.

Finally, call it by piping.

``` sh
ls | ./bo.sh
```

In this example, it will push a directory listing of the current directory.

![Notification received on mobile phone](/images/bo-sh/bosh03.png)

And again, on device we see:

![Notification received on mobile phone](/images/bo-sh/bosh04.png)

The script can be pulled from [Github](https://github.com/2bytes/Bashover)