By default, only certain metrics cover EC2 instances (host based metrics). However, at times we may want to monitor certain aspects of our operating system. EC2 provides scripts that we can install on the system to achieve this. To configure those scripts and ensure our EC2 instances have permissions to communicate with Amazon CloudWatch to put the metrics data into CloudWatch.

Commands used:

sudo yum install perl-Switch perl-DateTime perl-Sys-Syslog perl-LWP-Protocol-https

curl http://aws-cloudwatch.s3.amazonaws.com/downloads/CloudWatchMonitoringScripts-1.2.1.zip -O

unzip CloudWatchMonitoringScripts-1.2.1.zip

./mon-put-instance-data.pl --mem-util --mem-used --mem-avail --swap-util --swap-used --disk-space-util --disk-space-used --disk-space-avail --memory-units=megabytes --disk-space-units=gigabytes --disk-path=/dev/xvda1