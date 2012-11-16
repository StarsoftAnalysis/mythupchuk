mythupchuk -- MYTHtv script to UPdate CHannels for the UK
=========================================================

mythupchuk is a helper script for use with MythTV.

The problem is that it's difficult to line up channel names, numbers, icons
and scheduling information for broadcasts in the UK.

Various people have come up with SQL scripts and other methods of solving the problem.
This is my solution.

This may even be useful in countries other than the UK.

PLEASE backup your MythTV database (or at least the 'channel' table) before running this script: I've tested it, 
but I can't guarantee that it won't destroy your MythTV installation, or your whole computer for that matter.  
See the GPL3 licence for details.

How It Works
------------

mythupchuk combines information from a number of sources:

1. the mythtv database -- specifically the 'channel' table;
2. xmltvids from the output of the `tv_grabber_uk_rt` command;
3. icons from sites as such www.lyngsat-logo.com 
4. a user-edited file of channel details

Matching the file with the database is done by using fields sourceid and serviceid.
Normally this gives a unique match, but it is possible (if signals are received from
more than one transmitter, for example) for duplicates to occur: mythupchuk copes with this 
situation, and updates all matching channels.

Matching channel names in the file against the xmltvid is more tricky: some of the names in the xmltvid list
are different, and a few trick have to used to make them line up.  Firstly, names are converted to lower case
and all the spaces are removed.  Then the script applies a set of rules such as:

    return 'bbconeyorkshire&lincolnshire'   if /^bbc1yrks&lin$/;
    return 'bbconenorthernireland'          if /^bbc1ni$/;
    return "bbcone$1"                       if /^bbc1(.*)$/;    # BBC1 catch-all

Freeview channels sometimes don't include a region.  mythupchuk defaults to BBC 1 South,
BBC 2 England, and ITV Meridian.  These defaults can be overridden using the `--bbc1`, `--bbc2`,
and `--itv` options.

The xmltvid information also contains the icon urls that are used to obtain channel icons.

With the `--fileupdate` option, mythupchuk modifies the input file with up-to-date information from 
the database: new channels are added to the end of the file, and channels that no longer exist
are marked with a '#x' prefix which acts like a comment.  If a '#x'-commented channel reappears
(after retuning for example), the prefix is removed.

Note that the xmltvid and icon columns in the file are intend only to override the values from the tv\_grabber command: 
these values are not changed in the file.

With the `--dbupdate` option, then channel name, number, callsign, and visible flag are updated in the 
database from the values in the input file.  The xmltvid and icon fields for a channel are also 
updated from the tv\_grabber information, unless overridden by entries in the input file.

Options `--fileupdate` and `--dbupdate` can be used together, because different information is 
being transferred in each direction.

An example channel file is included -- the one currently used by the author, which may not
be to everyone's taste.  The channel numbering is non-standard, and the choice of which
channels to exclude is a personal one.  And it's for a system where source id 1 is Freesat,
and source id 2 is Freeview. *Do not update your database from this file
until you have edited it to meet your needs, and removed that deliberately-invalid
line at the end.*  It will give you an idea of what can be done, though.

Overview
--------

The following diagram summarises the flow of information when using mythupchuk.

                                                       .------------.  
                                                      (  TV Grabber  )
                                                       '------------'
                                                       /  
                                                    xmltvid and icon
    +-----------+       --dbupdate:                  /                        +------------+
    | Database  | <---- Update channel details <----'--- name, callsign, ---- | mythupchuk |
    | 'channel' |       Delete channels                  channum, etc.        |    input   |
    |  table    |                        --fileupdate:                        |    file    |
    +-----------+ ---------------------- Add new channels ------------------> +------------+  
                                         Mark non-existing channels 

Typical Usage
-------------

1. Set up MythTV, and tune in your tuners.  You'll probably end up with lots of channels in a not very useful order, and no schedule information.
2. Run 

        mythupchuk --fileupdate channels.list

where 'channels.list' is the name of a non-existent file that will be created.
3. Edit 'channels.list' (or whatever you called it), to set the channel numbers to your liking.  
You can also change the callsigns and names, but make sure that the name will still match the 
name of the corresponding xmltvid (see above).  Don't change the sourceid or serviceid column.  
Set the 'visible' column to 1 for channels that you don't want in the listings.  
Set the 'delete' column to 1 for channels that you want to get rid of completely.
4. Run

        mythupchuk --dbupdate channels.list

to check the file and see what changes would be made to the database.
5. When you're happy with the results, run

        mythupchuk --dbupdate --commit channels.list

to actually change the MythTV database.

You can re-run `mythupchuk --dbupdate [--commit]` at any time when you've made some more changes to the file.

When channels in the database have changed, because you've retuned for example, you can run
`mythupchuk --fileupdate channels.list`
which will add any new channels to the end of the file, and mark no-longer-found channels with the special
comment-like prefix '#x' to indicate their non-existence.

Deleting Channels
-----------------

Setting the delete flag for a channel, and then running `mythupchuk --dbupdate --commit` will delete
the channel from the database.  Note that there is no way to undelete a channel, other than retuning, because
the mythupchuk input file does not store all the information from the database.  Changing a channel's
delete flag back to '0' will not magically make the channel appear again, but will mean that the
channel does not get deleted again after retuning.

Xmltvids and Icons
------------------

In the UK, TV schedule information is available from Radio Times Ltd. via computer-readable listings 
which can be obtained using the the `tv_grabber_uk_rt` programme.  It produces a listing such as this
excerpt:

    <channel id="northern-ireland.bbc1.bbc.co.uk">
        <display-name>BBC One Northern Ireland</display-name>
        <icon src="http://www.lyngsat-logo.com/logo/tv/bb/bbc_one_northern_ireland.jpg" />
    </channel>
    <channel id="yorkshire.bbc1.bbc.co.uk">
        <display-name>BBC One Yorkshire</display-name>
        <icon src="http://www.lyngsat-logo.com/logo/tv/bb/bbc_one_yorkshire_north_midlands.jpg" />
    </channel>

which mythupchuk parses and matches against the channel names (see above).

mythupchuk runs the command like this:

    tv_grab_uk_rt --list-channels --config-file /dev/null 2> /dev/null

Note the use of `/dev/null` as the configuration file, which makes sure that it obtains details
of all possible channels, ignoring `~/.xmltv/tv_grab_uk_rt.conf`.

Licence
-------

mythupchuk is released under the GNU General Public Licence version 3 or later.  
This and any other documentation provided with mythupchuk
is licensed under the GNU Free Documentation Licence version 1.3 or later.

It has been written by Chris Dennis, chris@starsoftanalysis.co.uk.  Please feel free
to contact the author at that address with any questions or comments.

Version 0.3.2, 16 November 2012
