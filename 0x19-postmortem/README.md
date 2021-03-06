# 0x19. Postmortem

## Overview
The objective of this project was to write a postmortem for a system outage or system issue as practice. The postmortem was about a made-up issue using a standard postmortem format to document the issues faced, the resolution, the timeline, and preventative measures in the future.

-----------------------------------------------------------------

# Apache2 Postmortem (incident #504)
## Date: 
2018-09-12

## Authors:
Derek Kwok

## Status:
Complete, action items in progress

## Issue Summary
An outage was detected last week that lasted 1 day between the dates of 9/10 7:55 PT to 9/11 6:00 PT. The web server associated with our main website service was unable to serve the web content that was requested leading to customer complaints and a smaller traffic volume during that whole week. This outage affected 80% of users as other parts of the website located on different web servers behaved correctly. The root cause appears to be associated with the settings files on the Apache 2 web server application pointing to a nonexisting symlink. There appears to have been a misspelling in this settings file.

## Timeline
* 9/10 7:55 PT - First user complaint was filed showing that there were internal server errors in the HTTP response of code 500
* 9/10 8:30 PT - DevOps engineers' HTTP Testing appears to show that the web server associated with the customer portal in particular was unable to serve content
* 9/10 12:00 PT - Due to issues with more important projects relating to end-of-year launch goals, the issue was put on hold
* 9/10 15:00 PT - Local testing utilized STRACE while running an HTTP GET request to observe the system calls made by the program. STRACE was first used with the GET request command.
* 9/10 17:00 PT - Process ID's were used with STRACE to view the HTTP GET request and the resulting system calls in real time. Two process ID's were tested: 54 and 59.
* 9/10 19:00 PT - The issue was forwarded to the DevOps night crew on call
* 9/11 4:00 PT - The night crew found the underlying issue was associated with process 59 associated with the www-data process in Apache2
* 9/11 5:00 PT - Using real-time STRACE with a CURL GET request, the error log shows that the symbolic link called locale-wp.phpp did not exist. This was pointed to by the file wp-settings.php.
* 9/11 6:00 PT - The file wp-settings.php was pointing to locale-wp.phpp which was a typo. The actual file was locale-wp.php. This was fixed and CURL testing showed the issue was resolved.

## Root Cause
The issue was first noticed at 7:55 PT on Monday, 9/10. After performing several STRACE test calls on an HTTP GET request and observing the 500 status code of the response, it was observed that there was an issue in the settings and that some files did not exist and could not be found. The issue was caused by a typo in the settings file for Apache2 where the symbolic link filename did not exist. The wp-settings.php was pointing to locale-wp.phpp when the file that actually existed in the folder /var/www/html/locale/ was called locale-wp.php. The issue was fixed by going into the wp-settings.php file and correcting the typo from .phpp to .php on the file called locale-wp.php.

## Preventative Measures
Improvement efforts to reduce frequency of such issues is to have settings files looked at by several engineers to ensure that all symlinks are correct and are associated with files that exist on the local system. 

Action items:
* Patch apache2 every two weeks (1 day)
* Add monitoring tools to all web servers to web traffic to show dips in usage which could suggest outages are present (2 days)
* Frequently perform maintenance checks to ensure HTTP requests return expected responses (1 day)
* When settings files or administrative files are changed, perform a full test on web architecture to ensure nothing is broken (2 days)

## Supporting information
No supporting information at this time
