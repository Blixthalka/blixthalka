---
layout: post
title: Starting 2019 with some Bash scripting
---
The calendar has has incremented the year variable to 2019 and you still haven't acted on your new years resolution.
But don't worry, you don't have to go to the gym. 
Instead I propose that you switch your resolution to something else like, for instance, to streamline your workflow with Bash.
By learing some tricks in Bash, you will become much a more efficient developer and save lots of time in the long run. 
So, it's perfect that you have found this page, because I will show you a trick to access logfiles and hopefully it will give you some inspiration to write a script on your own. 

Accessing logs may be tideous.
First they may be located at far into the system, like  `/var/logs/program`.
We don't want to type in this every time we want to access the logs (yes, we have autocomplete, but that is a little bit to much work still). 
Secondly, there may be multiple different applications that we want to check the logs for.
Lets say we have logs names like `app1.log`, `app2.log` and `app3.log`, all located at the same place.
The last thing may be that the logs are dated and a new log is created for each day, we don't want to remeber what day it is today to access todays log. 
It may look like this `/var/logs/program/app1_2019-01-03.log`. 

To access todays log efficiently, we will create a bash function that we will put in `~/.bashrc` which will then be accessable in the shell.
The function is pretty straight forward and looks like this.

```bash
logf () {
        tail -300f /var/log/program/${1}_`date +"%Y-%m-%d"`.log
}
```

This will let us access todays logs of one specific app with `logf app-name`. 
Easy, right?
But how does this little function achieve this?
`tail` is a program that outputs the end of a file, passing the flags `-300f` will show the last 300 lines and wait for more input and output the new input if it comes.
Tail takes the log file as an argument, but to find the logfile some more stuff has to be done. 
First the static path to where the logs subside is inputted. 
After that we want to find the file, so we use `${1}` which is the first argument passed to the function, so it will be the app name. 
Then to get todays date we use the `date` function and passes a date formatting string to it, so it will align with the date format of the logfile. 

And we have our little efficient function here, which will maybe save us 5 seconds every time we access a logfile. 
It is not much, but if we add other small efficient functions like this, it will eventually start to add up to substantial amount of time.

Over and out.  
//Emil
