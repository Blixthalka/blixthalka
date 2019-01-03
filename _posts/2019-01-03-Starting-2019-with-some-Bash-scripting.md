---
layout: post
title: Starting 2019 with some Bash scripting
---
The calendar has has incremented the year variable to 2019 and you still haven't acted on your new years resolution.
But don't worry, you don't have to go to the gym. 
Instead I propose that you switch your resolution to something else, like for instance to streamline your workflow.
By learing some tricks in Bash, you will become much more efficient and save lots of time in the long run. 
So, perfect that you have found this page, because I will show you some tricks. 

First tip.
Accessing logs may be tideous.
First they may be at some far fetched path like, for instance, `/var/logs/program`.
You don't want to type in this every time you want to access the logs (yes, we have autocomplete, but that is a little bit to much work still). 
Secondly, there may be multiple different programs that you want to check the logs for.
Lets say you have logs names like `app1.log`, `app2.log` and `app3.log`.
The last thing may be that the logs are dated and a new log is created for each day, you don't want to remeber what day it is to access todays log. 
It may look like this `app1_2019-01-03.log`. 

To access todays log efficiently, we will create a bash function that we will put in `~/.bashrc` which will then be accessable in the shell.
The function is simple and looks like this:

```bash
logf () {
        tail -300f /var/log/program/${1}_`date +"%Y-%m-%d"`.log
}
```

This will let you access todays logs of one specific app with `logf app1`. Easy right?
But how does this little function achieve this?
`tail` is a program that outputs the end of the logfile, passing the flags `-300f` will show the last 300 lines and wait for more input and output the new input if it comes.
Tail takes the log file as an argument, but to find the logfile some more stuff has to be done. 
First the static path to where the logs subside is inputted. 
After that we want to find the file, so we use `${1}` which is the first argument passed to the function, so it will be the app name, like `app1`. 
Then to get todays date we use the date function and passes the date formatting string so it will align with the date format on the file. 

And we have our little efficient function here, which will save us maybe 10 seconds everytme we access the log. 
It is not much, but if we add it up during your whole carrer it will be substantial amount of time.  

Over and out.
