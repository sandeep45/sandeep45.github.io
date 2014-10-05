---
layout: post
title: RDS vs Managed Databse on EC2 via Cloud66
published: True
categories: []
tags: []
---

# Amazon RDS

For our application, I think a memory optimized r3.2xlarge is a good choice. It got 8 vCPU's, 61GB of RAM and PIOPS optimized.

Its on demand cost is $0.995 per hour. i.e. $716 for a 30 day month.

![rds pricing](/assets/post3/rds_cost.png)

# Amazon EC2
If we were to do this ourselves, I would go with a memory optimized r3.2xlarge  with 8 vCPU's and 61GB of RAM, it comes with 1 SSD of 160GB.

Its on demand cost is $0.700 per hour. i.e. $504 for a 30 day month.

![rds pricing](/assets/post3/ec2_cost.png)

Now to do this myself, I am going to use [cloud66](http://www.cloud66.com) to do the heavy lifting of setting up and maintaing the DB. Lets look at its associated costs.

Cloud66 will cost me an additionaly $9 for the server. Now I also want their managed backups. That would cost me an additional $12 per month + $0.12 per GB. Assuming a 100GB Databse size that's an additional $12.

![rds pricing](/assets/post3/cloud66_backup_pricing.png)

So total I am out 504 + 9 + 12 + 12 = $537

# Conclusion

So setting up DB manually on an EC2 instance along with cloud66 support gives the same level of comfort as doing it on RDS gives. Its also cheaper when going the EC2 + cloud66 route. Additionally, with cloud66 I have all my architecture being built and managed at one spot. So in theory tomorrow if i want to ditch AWS and go to [rackspace](http://www.rackspace.com) its easier for me to pack up and just move as all my belongings are in space - cloud66.

**The verdict**: EC2 + cloud66 wins over RDS.

