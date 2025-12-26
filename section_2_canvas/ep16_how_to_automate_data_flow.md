Steps to automate data flow:

1. create data flow in data wrangler
2. click on last step of data processing > export > add destination > s3
3. click on ... of s3 destination > export > untoggle auto job config > associate schedules

in `associate schedules`, you can setup your schedule to automatically run the data flow as a job