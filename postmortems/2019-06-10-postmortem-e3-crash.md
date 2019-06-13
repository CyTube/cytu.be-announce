# Postmortem of June 9 (E3) outage

## Non-technical summary

Due to a combination of a bug in software used by CyTube and additional user
traffic for watching E3, one of CyTube's servers exceeded the maximum number of
connections and became unavailable for a period of about 11 minutes.  Since this
server was also hosting the website, all CyTube users were unable to access any
channel during this time.

## Timeline

  * *2019-06-09 20:00 UTC* - CyTube server breaches `RLIMIT_NOFILE` of 4096;
    some operations start logging `EMFILE` errors
  * *2019-06-09 20:05 UTC* - The last log line is flushed to disk.  Beyond this
    point, the logs are unavailable because the process became completely
    unresponsive
  * *2019-06-09 20:06 UTC* - Monitoring data indicates the website is down
  * *2019-06-09 20:09 UTC* - First user report (in IRC) that the website is down
  * *2019-06-09 20:10 UTC* - Unsuccessful attempt to gracefully reboot the
    service
  * *2019-06-09 20:12 UTC* - Channel traffic is redirected to a healthy server
    and the webserver is hard reset
  * *2019-06-09 20:13 UTC* - Website begins serving traffic again

## Technical details

Under normal circumstances, CyTube's servers should not be allocating more than
about 2,000 file descriptors (accounting for user sockets as well as log files,
database connections, etc.), less than half the default rlimit of 4096.
Monitoring data collected after the event indicates that in the weeks leading up
to the crash, the daily peak of open FDs on this server was increasing (the
linear regression shows approximately 50 FDs/day), indicating a possible FD
leak.

After further investigation, one source of leaked FDs was determined to be the
library CyTube uses for sending password reset emails, which leaks 1 FD per
email sent (CyTube doesn't use an SMTP connection pool because emails are
infrequently sent): https://github.com/nodemailer/nodemailer/pull/990

However, this leak alone didn't account for all the orphaned sockets being
identified.  Further investigation was done and determined that in specific 10.x
LTS versions of node.js, TLS sockets corresponding to websocket connections can
be leaked when the socket is closed unexpectedly, e.g. due to a network issue.
The latest version does not appear to have this problem.

## Followup

A few efforts are underway to help diagnose and prevent this issue going
forward:

  1. Deploy updated versions of node.js and nodemailer that resolve the FD leaks
  2. Add monitoring for open FDs to detect when a server is close to the limit
  2. Separating the webserver from the user socket servers so that in the event
     of one node crashing, it doesn't also take down the website
