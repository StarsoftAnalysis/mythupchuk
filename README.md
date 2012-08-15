mythupchuk -- MYTHtv script to UPdate CHannels for the UK
=========================================================

mythupchuk is a helper script for use with MythTV.

The problem is that it's difficult to line up channel names, numbers, icons
and scheduling information for broadcasts in the UK.

Various people have come up with SQL scripts and other methods of solving the problem.
This is my solution.

This may even be useful in countries other than the UK


For initial testing, it's a good idea to use a test version of the mythtv
database, which can be created with a simple script such as:
	# mysqladmin create DB_name -u DB_user --password=DB_pass && \
	  mysqldump -u DB_user --password=DB_pass DB_name | mysql -u DB_user --password=DB_pass -h DB_host DB_name
(thanks to http://www.rubyrobot.org/article/duplicate-a-mysql-database)
Then set the --mysqltxt option to use the details of the test database.

mythupchuk uses 'sourceid' and 'serviceid' to uniquely identify a channel (although chanid is the table's primary key).  
It does that because
serviceid is a real value that relates to the channel
(see the SID column on http://www.lyngsat.com/Astra-2B.html for example)
rather than something cobbled together by MythTV.

Typical Usage
-------------

1. Set up MythTV, and tune in your tuners.  You'll probably end up with lots of channels in a not very useful order, and no schedule information.
2. Run 
		mythupchuk --fileupdate channels.list
where 'channels.list' is the name of a non-existent file that will be created.
3. Edit 'channels.list' (or whatever you called it), to set the channel numbers to your liking.  You can also change the callsigns and names, but make sure that the name will still match the name of the corresponding xmltvid (see below).  Don't change the sourceid or serviceid column.  Set the 'visible' column to 1 for channels that you don't want in the listings.  Set the 'delete' column to 1 for channels that you want to get rid of completely.
4. Run
		mythupchuk --dbupdate channels.list
to check the file and see what changes would be made to the database.
5. When you're happy with the results, run
		mythupchuk --dbupdate --commit channels.list
to actually change the MythTV database.

You can re-run 'mythupchuk --dbupdate [--commit]' at any time when you've made some more changes to the file.

When channels in the database have changed, because you've retuned for example, you can run
		mythupchuk --fileupdate channels.list
which will add any new channels to the end of the file, and mark no-longer-found channels with the special
comment-like prefix '#x' to indicate their non-existence.

XMLTVID
-------

[more stuff needed here]

Licence
-------

mythupchuk is released under the GPL3 licence.

It has been written by Chris Dennis, chris@starsoftanalysis.co.uk



