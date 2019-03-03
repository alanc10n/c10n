---
title: "S3 Outage"
date: 2017-03-04T17:21:10-07:00
draft: false
---
Wednesday morning I, along with operations folks all over the country, started getting alerts from systems that depend on Amazon's S3 object storage. Quarqnet uses S3 to store uploaded data from athletes using our Qollector. We also depend on S3 to store some files that we occasionally read in order to provide our services. S3 is one of Amazon's oldest and most reliable services, and a great many online systems depend on S3 to store data. The typical advice for AWS services is to make sure they're configured to operate in multiple availability zones. Since S3 is, by default, configured to operate in multiple AZs, I've always taken for granted that the service would be available, which was of course foolish.

## Oh no

Around 10:30 MST, all requests to S3 stopped responding, bringing our services to a grinding halt. It took a while for me to believe that the problem was on Amazon's side and that S3 was really down, but a quick check of Twitter showed a lot of people running around with their hair on fire. Eventually it became clear that the entire US-East-1 region was in trouble (AWS divides their services into regions and further into availability zones within each region). I did some quick thinking about trying to work around the outage, but decided it wasn't worth the trouble and instead waited for service to be restored, which I assumed would happen promptly. Again, I was foolish. It was almost four hours before our write requests began reliably working.

## What about next time?

Given this experience, I've realized that we need a plan for extended S3 outages in our region. Since our workload is write-heavy, it's easier to come up with a solution to keep running after losing access to our primary buckets. Several options involve using [Cross-region Replication](http://docs.aws.amazon.com/AmazonS3/latest/dev/crr.html) to mirror writes in one bucket to a different bucket in another region. Note that the S3 bucket namespace is global, meaning that only one bucket can exist with a given name. Although an S3 outage in a given region would likely prevent replication from happening immediately, eventually objects written to the source would appear in the destination bucket.

So, if you have backup buckets in another region, in the event of an outage it would be possible to point your apps at the backup bucket and continue operation. Keep in mind, of course, that latency to a bucket in a different region is likely to be substantial, so if you're only maintaining S3 buckets in the other region but keeping the rest of your service running in the primary region, you'll need to be able to tolerate extra latency in the hundreds of milliseconds. Given that S3 is not the only service that can fail, the most reliable architecture involves being able to spin up all components of your system in another region: EC2, RDS, etc. It might be cost-prohibitive to keep these resources running, but having the ability to bring them online in a reasonable amount of time could turn your next four-hour outage into a 10 minute one.
