---
layout: post
title: Move Logs on AWS to a different drive
published: True
categories: [DevOps]
tags: [AWS, Logs, Rails Logs, Cloud66 Logs]
---

I am using [Cloud66](http://cloud66.com) to manage my servers which are built on EC2 Instances at [AWS](http://aws.amazon.com). I have some pretty big boxes - c3.2xlarge

![aws instance size c3.2xlarge](/assets/post2/aws_instance_size.png)

The specs on these box and pretty nice:

![aws instance spec](/assets/post2/ec2_spec.png)

But for some reason `df` on my box shows:

![df](/assets/post2/df_on_instance.png)

 **Note**: I configured these boxes directly from [cloud66](http://cloud66.com) and this is how they are setup. My entire application is in `/` which is mounted on `/dev/xvda1` and I have a big empty folder at `/mnt` which is mounted on `/dev/xvdb`.


Cloud66 Engineer [Philip](https://www.linkedin.com/in/philipkallberg) explained to me that is correct and this is what AWS gives:

![c66 aws default](/assets/post2/cloud66_explanation1.png)

After my rails application is installed, gems are setup, plugins, imagemagick etc. etc. I only have ~3GB left on primary drive. This is not a lot of space. My logs alone on busy days are greater than 3GB.

I found that i have two sets of logs which grew very quickly on busy days:

1. `$STACK_PATH\log\unicorn.*.log` (Rails Server Logs)
2. `\var\log\nginx\access.log` (Nginx Logs)

The goal of the rest of this post is to **document** the steps I took to move these logs to my `/mnt` 80GB _ephemeral_ drive. Ephemeral means that if you change the instance type for the server you will lose it.

1.  Setup new log directories:

    ```
    sudo su -
    cd /mnt
    mkdir log
    cd log
    mkdir nginx
    mkdir web
    # Create empty log files for unicorn as it will complain with files being present
    cd /mnt/log/web
    echo "" > unicorn.stdout.log
    echo "" > unicorn.stderr.log
    # ease out permissions as various process being run by various users need to access them:
    chmod 777 -R /mnt
    ```


2.  Make Nginx send logs to the new directory

    ```
    # /etc/nginx/cloud66_nginx.conf
    error_log  /mnt/log/nginx/nginx_error.log;
    access_log /mnt/log/nginx/access.log varnish_log;
    ```

3.  Make Unicorn send logs to the new directory

    ```
    # $STACK_PATH/config/unicorn.rb
    stdout_path "#{ENV['LOG_PATH']}/web/unicorn.stdout.log"
    stderr_path "#{ENV['LOG_PATH']}/web/unicorn.stderr.log"
    ```

4.  Add Env Variable which has location of new log directory

    ```
    LOG_PATH=\mnt\log
    ```

5.  Create new Log Rotation File

    ```
    cd /etc/logrotate.d
    ln -s $STACK_PATH/config/mnt.conf ./mnt
    ```

6.  Build `mnt.conf` in Rails App so it can be commited and tracked

    ```
    /mnt/log/*/*.log {
      # compress the log files
      compress

      #rotate logs daily
      daily

      #keep two weeks of logs
      rotate 14

      #set the location of the old log files
      noolddir

      # don't throw an exception if a log file is missing
      missingok

      # don't rotate if it is empty
      notifempty

      # set extension of rotated log files
      dateext
      dateformat %Y-%m-%d.
      extension log

      # truncate the file, allowing processes to still write to the same handle
      copytruncate

      # create log file after rotation
      create

      # run the scripts only once regardless of the number of files being rotateds
      sharedscripts
      prerotate
          if [ -d /etc/logrotate.d/httpd-prerotate ]; then \
                  run-parts /etc/logrotate.d/httpd-prerotate; \
          fi \
      endscript
      postrotate
        [ ! -f /var/run/nginx.pid ] || kill -USR1 `cat /var/run/nginx.pid`
      endscript
    }
    ```

7. Redeploy and Restart everything

    ```
    sudo /etc/init.d/nginx restart
    sudo bluepill cloud66_web_server
    ```

#The setup is complete, now lets test.

After the servers are back up and some traffic has been sent to the servers new log files should appear

![nginx ls](/assets/post2/nginx_log_ls.png)

![nginx ls](/assets/post2/unicorn_log_ls.png)

Now we can force a logrotate and see if that part is also working

```
cd /etc/logrotate.d/
sudo logrotate -f mnt
```

Jumping back to the logs folder we should see rotated logs

![nginx ls](/assets/post2/nginx_gzipped_log_ls.png)

![nginx ls](/assets/post2/unicorn_gzipped_log_ls.png)

# And we are Done - Logs have now been happily moved!

