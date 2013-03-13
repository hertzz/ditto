# Ditto-Backup #
A simple bash driven MySQL and local file backup system for archiving files to Amazon AWS Simple Storage Service.

1. Installation
2. Configuration
3. Troubleshooting

#### 1. Installation ####
To get started installing Ditto, you will need to clone <http://www.github.com/base2Services/ditto.git>  to the target host.

	$ mkdir -p /opt/base2
	$ cd /opt/base2
	$ git clone https://www.github.com/base2Services/ditto.git

Next, you need to initialize Ditto so that it will automatically run scheduled actions.

	$ /opt/base2/ditto/bin/ditto -i

By running the above command, Ditto will automatically install a cronjob in **/etc/cron.d**. The cronjob itself runs as the root user, however you can change this by modifying **/etc/cron.d/ditto-backup.cron**. The cronjob makes Ditto execute every minute continuously which will fire off the boostrap process and procedurally trigger any actions defined in the schedules directory (presuming they are allowed to run at that point in time).

#### 2. Configuration ####

#####MySQL backups#####

If you need to backup one or more databases, you will need to configure a MySQL schedule for Ditto which will periodically back up your important data. To start, you can either create a new MySQL schedule, or you can edit the existing schedule stub which has been provided for you.

Goto the Ditto schedules directory:

	$ cd /opt/base2/ditto/schedules
	
If creating a new MySQL schedule, create a file in the above directory which describes the action in a way you can easily determine what it's doing, prefixed with a 3 digit number and prepended with **".mysql**".

	$ touch 003-all-production-databases.mysql
	
If you want to use the existing stub, open the default schedule to edit it:

	$ vi 001-all-databases.mysql
	
The number prefix before the action description is used to process schedules in a specific order. Files do not need to have a unique number in their filename.

The **".mysql"** file extension tells Ditto to use the MySQL action handler to deal with the schedule.

To get the blank schedule running, you will need to add the action instructions into the file which you created above:

	$ vi /opt/base2/ditto/schedules/003-all-production-databases.mysql
	
Add the following content into the schedule:

	#!/usr/bin/env bash
	#

	WHEN="0 21 * * *"

	MYSQL_USERNAME=username
	MYSQL_PASSWORD=password
	#MYSQL_DATABASES=all
	#MYSQL_HOSTNAME=localhost
	#MYSQL_GZIP_DATA=false
	#MYSQL_DATED_DUMP=true
	#MYSQL_HOTCOPY=false

You will need to change the Username and Password properties, however all of the other settings will work as their defaults.

To back up individual databases, change the databases property to a space seperated list of database names in double quotes.

The "**WHEN**" property is used to tell the schedule when it is allowed to execute. The string specified for WHEN refers to a dumbed down version of a regular cron expression. For example, if you wanted the MySQL schedule to run at 02:25 PM daily, you would specify the following:

	WHEN="25 14 * * *"
	
The expressions, however, **do not support advanced operators**. Basically, you can only use single or double digit numbers for all of the values. An asterisk implies that it should always run for that specific field. (See the cron man page for more information about the field orders).

#####S3 backups (Simple storage service)######

If you want to backup files to S3, you will need to configure an S3 schedule for Ditto which will periodically push your important data to the cloud. To start, you can either create a new S3 schedule, or you can edit the existing schedule stub which has been provided for you.

Goto the Ditto schedules directory:

	$ cd /opt/base2/ditto/schedules

If creating a new S3 schedule, create a file in the above directory which describes the action in a way you can easily determine what it's doing, prefixed with a 3 digit number and prepended with **".mysql**".
	
	$ touch 900-backups.s3
	
If you want to use the existing stub, open the default schedule to edit it:

	$ vi 999-backup.s3
	
The number prefix before the action description is used to process schedules in a specific order. Files do not need to have a unique number in their filename.

The **".s3"** file extension tells Ditto to use the S3 action handler to deal with the schedule.

To get the blank schedule running, you will need to add the action instructions into the file which you created above:

	$ vi /opt/base2/ditto/schedules/900-backups.s3
	
Add the following content into the schedule:

	#!/usr/bin/env bash
	#

	WHEN="0 23 * * *"
	
	S3_ACCESS_KEY=""
	S3_SECRET_KEY=""
	S3_BUCKET_URI=""
	S3_USE_ENCRYPTION=false
	S3_USE_COMPRESSION=false
	#S3_MD5_CHECKING=true
	#S3_REMOVE_DELETED=false
	#S3_PRESERVE_PERMISSIONS=true
	S3_MAX_CPU=75

You will need to set the access key, secret key and bucket URI properties, however all of the other settings will work as their defaults.

When the S3 schedule runs, it will automatically synchronize all files from the Ditto backups directory to the specified Bucket location which by default is **/opt/base2/ditto/backups**.

You can override the path which the schedule is using as the source location by configuring the following property:

	S3_BACKUP_DIR=/path/to/my/dir

The "**WHEN**" property is used to tell the schedule when it is allowed to execute. The string specified for WHEN refers to a dumbed down version of a regular cron expression. For example, if you wanted the S3 schedule to run at 04:45 PM daily, you would specify the following:

	WHEN="45 16 * * *"
	
The expressions, however, **do not support advanced operators**. Basically, you can only use single or double digit numbers for all of the values. An asterisk implies that it should always run for that specific field. (See the cron man page for more information about the field orders).

#### 3. Troubleshooting ####

All logging information is by default, written to syslog using logger. On RedHat/CentOS systems, you can look at what Ditto is doing using the following command:

	$ tail -f /var/log/messages | grep DITTO
	
On Debian based systems such as Ubuntu, you can run the following command:

	$ tail -f /var/log/rsyslog | grep DITTO