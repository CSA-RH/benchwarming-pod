# Scaling deployments with cron jobs

Say you have a deployment running, but you notice that there is not that much demand for it at night, so you decide you will scale down de number of replicas every day at 21:00 to save some money and scale up at 08:00 so you can sustain higher bursts in pod usage.

There are many ways to accomplish this, some more sophisticated than others, but a very simple method is just to use cron jobs.

These jobs will be run on the specified periocity, either hour based (for example at 08:00 every day) or by ellapsed time (every 2 minutes).

## Setup


