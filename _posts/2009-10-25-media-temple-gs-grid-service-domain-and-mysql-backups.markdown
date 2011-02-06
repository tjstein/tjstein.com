---
layout: post
title: Media Temple (gs) Grid Service Domain &amp; MySQL Backups
excerpt: 2 crons, 1 cup...
comments: true
---

Having been on many shared hosting platforms before, I am fortunate enough to have a few (gs) Grid Services with (mt) Media Temple. Although no native backup tool currently exists for the (gs), I've created some simple backup scripts to backup the domain directories and MySQL databases. Both are very simple bash scripts that can be set up as cron jobs from with the AccountCenter. The first backup we'll create is for domain backups:

<ol>
	<li>If you haven't done so already, <a href="http://kb.mediatemple.net/questions/16/" target="_blank">enable SSH</a> from the (mt) Media Temple AccountCenter.</li><br>

	<li>Once you've logged in via SSH, navigate to the /data directory. Keep in mind you'll need to add your site number in the 00000 for this to work. You can find your site number in the Server Guide &gt; Access Domain section of the AccountCenter.</li>

{% highlight bash %}
cd /home/00000/data
{% endhighlight %}

	<li>Run the following commands to create and set permissions of the backup directory:</li>

{% highlight bash %}
mkdir backup
chmod 777 backup
{% endhighlight %}

	<li>Create a file called backup.sh and make it executable:</li>

{% highlight bash %}
touch backup.sh
chmod +x backup.sh
{% endhighlight %}

	<li>Open the file in a text editor or vim and paste in the following:</li>
</ol>

{% highlight bash %}
#!/bin/bash
today=$(date '+%d_%m_%y')
echo "* Performing domain backup..."
tar czf /home/00000/data/backup/example.com_"$today".tar.gz -C / home/00000/domains/example.com
# Remove backups older than 7 days:
MaxFileAge=7
find /home/00000/data/backup/ -name '*.gz' -type f -mtime +$MaxFileAge -exec rm -f {} \;
echo "* Backed up..."
{% endhighlight %}

This really simple script creates a tar.gz backup of the example.com directory, stamps it with the current date and places it in the the backup folder we just created. It will also prune any backup older than 7 days old. This number can be adjusted by tweaking the <em>MaxFileAge</em> value. Before saving the file, be sure to make the necessary adjustments for your site number after the /home directory. You'll also need to modify example.com for the actual domain you'll be backing up. Before adding it as a cron, run the script via SSH -- the output should look like this:

<pre class="terminal">
example.com@n10:/home/00000/data$ ./backup.sh
* Performing domain backup...
* Backed up...
</pre>

Depending on the size, the backup may hesitate after the initialization while the domain directory is being compressed. You can now verify that the new backup is in the <strong>/home/#####/data/backup/</strong> directory with the today's date: <em>example.com_25_10_09.tar.gz</em>. Once you've confirmed the backup works, add the backup.sh file as a cron within the AccountCenter following this <a href="http://kb.mediatemple.net/questions/243/" target="_blank">KnowledgeBase</a> article. You can schedule the cron as frequent as you would like.

Now, you've probably got at least 1 MySQL database that should be backed up as well. This can be accomplished just as easily.

SSH back into the /data directory and create a new bash script file:

{% highlight bash %}
touch db-backup.sh
chmod +x db-backup.sh
{% endhighlight %}

Now, open the file in a text editor or vim and paste in the following:

{% highlight bash %}
#!/bin/sh
#############################
SQLHOST="internal-db.s00000.gridserver.com"
SQLDB="db00000_dbname"
SQLUSER="db00000"
SQLPASS="db_user_password"
SQLFILE="db00000_example.com_$(date '+%d_%m_%y').sql"
LOCALBACKUPDIR="/home/00000/data/backup"
#############################
echo "* Performing SQL dump..."
cd $LOCALBACKUPDIR
mysqldump -h $SQLHOST --add-drop-table --user="$SQLUSER" --password="$SQLPASS" $SQLDB > $SQLFILE
# Remove backups older than 7 days:
MaxFileAge=7
find /home/00000/data/backup/ -name '*.sql' -type f -mtime +$MaxFileAge -exec rm -f {} \;
echo "* Backed up..."
exit 0
{% endhighlight %}

This bash script uses mysqldump to dump a MySQL database which stamps it with the current date and places it in the the backup folder we created before. This will also prune any database backups that are older than 7 days. Before saving the file, be sure to make the necessary adjustments for your site number and database credentials -- all of this can be found under the Manage Databases section of the AccountCenter. Before adding it as a cron, run the script via SSH -- the output should look like this:

<pre class="terminal">
example.com@n10:/home/00000/data$ ./db-backup.sh
* Performing SQL dump...
* Backed up...
</pre>

Once you've confirmed the script completes without errors, add it as a cron job so you'll never have to worry about losing your MySQL data again!