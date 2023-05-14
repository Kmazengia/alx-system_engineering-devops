Postmortem - sales and distribution system  Failure (incident #30)
![IsItDown Logo](https://github.com/Kmazengia/alx-system_engineering-devops/blob/master/0x19-postmortem)

## Date and Duration of Outage

2023-04-13 / 5:00 PM - 8:15 PM

## Authors

* kefale

## Status

Resolved, action items in progress

## Summary

IsItDown, our very popular and very useful site to through which 10 distributors are accessing there physical and 
logical inventorys, are down, went down. A 100 Internal Server Error disabled access for our users for about 3 hours 
while the issue was resolved. The irony was broadcast on linkedin in similar form and tone to the following one.
 An 3 hours of server debugging led to a resolution.
![IsItDown Logo](https://github.com/Kmazengia/alx-system_engineering-devops/blob/master/0x19-postmortem)

### Impact

100% of users experienced outages in the form of a 100 Internal Server Error. The whole user-facing site was unable to be reached. 
Estimated 300M queries lost, $100M of ad revenue foregone or requested back by companies whose ads were unable to be seen.

### Root Cause

100 Internal Server Error coming from java  section of
(Linux Apache Tomcat) stack. Settings file contained a typo
which caused a **File Not Found** error on thejav backend. It
was nearly invisible from the apache server because Apache
is not equipped with alerting capabilities to signal issues
with the content it was serving.

## Trigger

Any call to the server would trigger it due to the fact the java
halted by the error whenever it attempted to run.

## Resolution

The error was resolved through a simple typo fix. However, the
debugging process took the better part of 3 hours. A 100 error
means anything can be wrong with the server including the files and databases
it pulls from. Therefore, the first step was to the check the apache logs.
Apache only showed being started and stopped which is normal also indicating
that apache is writing to the logs in fact. Next, MySQL was checked. Databases
can go down and cause issues. Unfortunately, no issue could be found in the
MySQL logs either. Therefore, the problem was expected to be in the java code.
To debug java the debug mode for the java config was turned on. This changed the
server's response from 500 to 200. Great news, the problem is fixed, right?
Wrong. The server returned 200, but not because it was serving content. It
was rather serving something else. The good news is it served us the error in
the java files. It was a typo. On line 98 of the `wp-settings.java` file, a
require directive was referencing `class-wp-locale.javac`. This file did not
exist. Why? Its extension was wrong. All java files have a `java` extension.
`.javac` is a silly oversight in typing the correct file name. Once the typo
was resolved the whole server was serving the correct website content.

## Detection

Borgmon experienced a 100 Internal Server error when trying to access our site
for his own personal inquiry into the status of another site.

## Corrective and Preventative Action Items

| Action Item | Type | Owner | Bug |
| ----------- | ---- | ----- | --- |
| Setup automatic testing through Travis CI in github | mitigate | Ababe | n/a **DONE** |
| Develop code review framework which includes local test | prevent | Ayele | Bug 5554823 **TODO** |
| Develop dummy site for 500 Internal Server redirect, potentially using other website status apis for info rather than ours | alternative action |solomon | n/a **TODO** |
| Use development site for real testing prior to production deployment| prevent | Abebe | Bug 5554824 **TODO** |
| Facilitate communication between SRE and backend departments for more robust development | prevent | keleme | Bug 5554825 **DONE** |
| Deploy updated wordpress site to prod | prevent | Abebe | n/a **DONE** |
| Develop monitoring and detection tools for quicker detection of issue | mitigate | SRE Team | n/a **TODO** |

## Lessons Learned

* Never push untested/poorly tested code into production no matter how small the change

### What went well

* Debugging techniques were solid and used well in conjunction with development skills
* Production fix was quick

### What went wrong

* The debugging took too long
* Detection should have been much quicker

### Where we got lucky

* Linkedin was a quick means of getting feedback from our users and someone was scrolling linkedinr instead of working

## Timeline

2023-05-14 (*all times PST*)

| Time  | Description |
| ----- | ----------- |
| 14:51 | Team comes back from happy hour at "Mr.Anuar" on Addis|
| 15:53 | Team pushed code after a few modifications and code reviews |
| 15:54 | **OUTAGE BEGINS** -- Server is serving 100 Internal Server Error |
| 17:55 | solomon receives pager storm, `ManyHttp500s` from all clusters |
| 16:57 | IsItDown is no longer informing users of site statuses because it is itself down |
| 16:58 | solomon starts investigating, finds server error is indeed what the user sees |
| 17:01 | **INCIDENT BEGINS** docbrown Abebe incident #30 |
| 17:02 | someone coincidentally send linkedin message to us about the irony of our situation |
| 17:06 | Solomon investigates apache logs |
| 17:07 | SOlomon investigates mysql logs |
| 17:10 | Abebe investigates php in debug mode |
| 17:18 | Ayele finds file where error is originating from |
| 17:20 | Abebe fixes typo in file |
| 17:35 | Production push of code and server succeeds after error fix |
| 17:36 | **OUTAGE MITIGATED**, Serving accurate webpage to all user-facing pages now |
| 17:55 | Conducted final check on stability of server and determined it is ok to be allowed to go |
| 18:00 | **OUTAGE ENDS**, all traffic balanced across all clusters |
| 18:15 | **INCIDENT ENDS**, reached exit criterion of 30 minutes nominal performance |
