SyncDB v1.0
===========

by JP Lew (<http://jplew.com>)

SyncDB is bash deploy script meant to take the tedium out of synchronizing local and remote versions of a Wordpress site. It allows developers working in a local environment (eg. MAMP) to rapidly "push" or "pull" changes to or from their production server with a single terminal command. 

SyncDB synchronizes your sites by executing a sequence of shell commands. It uses `mysql` for your database, `rsync` for your uploads folder, and leaves it up to you to synchronize your themes, plugins, etc (I use `git` for this).

Search and replacing the site URLs in the database is handled by the CLI version of InterConnect IT's invaluable "Wordpress Search and Replace Tool" (<http://interconnectit.com/products/search-and-replace-for-wordpress-databases/>). 

After initial script configuration, you'll be able to migrate your entire Wordpress database and uploads folder by running SyncDB from the command line like so:

    ./syncdb

No more FTP programs, no more PHPMyAdmin, no more tears. SyncDB don't mess around.



Usage
=====

1. Copy syncdb to the root of your site and make sure it's executable (`chmod +x syncdb`).
    + This is important, as the script must be located in the same folder as your wp-config.php (and your local-config.php, if you're doing it the smart way ([Mark Jaquith's WP-Skeleton](http://markjaquith.wordpress.com/2012/05/26/wordpress-skeleton/).)

1. 
2. Configure the script
    + This step involves manually editing the variables located at the top of the script using your favorite text editor. This has to be done once for each local/remote site pair. All the settings are documented via comments in the script itself.

3. Run syncdb command from the command line: `./syncdb [command]`
> SyncDB works in either direction. It can "push" your changes to the remote server, or "pull" updates from it.
>
> To push, run:
>    `./syncdb`
>    *(same as `./syncdb push`)*
>
> To pull, run:
>    `./syncdb pull`
>
> For greater control, you can limit SyncDB to execute only the command of your choosing by passing the function name as an argument. See **List of Commands** below.



List of Commands
================

Below is a list of all of SyncDB's commands. They are meant to be passed one at a time like so: `./syncdb backup_local_db`. All commands are meant to be executed locally, unless noted *(remote only)*. 

   + `push` *(default)*
     + Execute the following sequence of commands:
         + test_ssh()
         + backup_local_db()
         + upload_local_db()
         + upload_script()
         + do_login_backup()
         + do_remote_operations()
         + do_search_replace_remote()
         + rsync_push()

   + `pull`
     + Execute the following sequence of commands:
         + test_ssh()
         + upload_script()
         + do_login_backup()
         + download_remote_db()
         + backup_local_db()
         + replace_local_db()
         + search_replace_local()
         + rsync_pull()

   + `test_ssh`
     + Check if the SSH connection is working.

   + `backup_local_db`
     + Back up the local MySQL database to latest-local.mssql.bz2 (or whatever you set $l_db_name to). Another backup dump is created with its name prepended by the current date and time. For example 20130902-1230-database.mssql.bz2.

   + `upload_local_db`
     + Upload the most recent local database dumpfile to the remote server.

   + `upload_script`
     + Upload a copy of SyncDB to your remote host's root directory.

   + `do_login_backup`
     + Login to the remote server via SSH and call backup_remote_db(), which will backup the remote MySQL database.

   + `backup_remote_db`
     + *(remote only)* Backup the remote MySQL database to latest-remote.mssql.bz2 (or whatever you set $r_db_name to). Another backup dump is created with its name prepended by the current date and time. For example 20130902-1230-database.mssql.bz2.

   + `do_search_replace_remote`
     + Login to remote server and execute search_replace_remote().

   + `download_remote_db`
     + Download the most recent remote database dump file via `scp`.

   + `replace_local_db`
     + Drop then recreate the local database. This effectively deletes all the tables.

   + `do_remote_operations`
     + Login to remote server and execute replace_remote_db() method.

   + `replace_remote_db`
     + *(remote only)* Drop and recreate remote database. This effectively deletes all the tables.

   + `search_replace_local`
     + *(remote only)* Download the Search and Replace tools locally then execute them.

   + `search_replace_remote`
     + Download the Search and Replace tools on the remote server, execute them, then delete them.
 
   + `rsync_pull`
     + Synchronize the local uploads folder with the contents of your remote uploads.

   + `rsync_push`
     + Synchronize the remote uploads folder with the contents of your local uploads 



How it works
============

SyncDB synchronizes the local and remote versions of your site's MySQL data and media files by executing a sequence of bash commands, outlined below.
 
** Remote Sync (`syncdb push`) **
   1. dump local database
   2. upload local database dump file to remote server
   3. dump remote database
   4. drop remote database
   5. recreate empty remote database
   6. fill remote database with content from local dump file
   7. install Interconnect IT search & replace scripts
   8. search and replace remote database with your remote url
   9. delete search & replace scripts
  10. upload your newest local media to remote FTP

** Local Sync (`syncdb pull`) **
   1. dump remote database
   2. ddownload remote database dump file
   3. dump local database
   4. drop local database
   5. recreate empty local database
   6. fill local database with content from remote dump file
   7. install Interconnect IT search and replace scripts
   8. search and replace local database with your local url
   9. download latest media from remote FTP

SyncDB does not sync your code. To achieve that, I run a separate `git` command (see "Complete workflow" below.



Complete workflow
=================

I wrote this script because I am constantly migrating Wordpress sites and needed a simple tool to automate the process. This script, therefore, plays one part in my overall Wordpress development workflow. Below is my set-up:

*** Local **
 * Mac OS X running MAMP (<http://www.mamp.info>) *[optional]*
 * Name-based virtual hosting enabled (eg. http://dev.mywebsite.com as opposed to localhost/mywebsite) *[recommended]*

*** Remote ***
 * SSH enabled *[mandatory]*
 * Password-less login with SSH Keys enabled (a guide for those unfamiliar: <http://sniptools.com/mac-osx/save-ssh-password-in-terminal>) *[optional]*
  
*** Common ***
 * Git-managed code using Joe Maller's awesome "Web-focused Git workflow" (<http://joemaller.com/990/a-web-focused-git-workflow/>) *[optional]*
 * Wordpress installation in WP-Skeleton repo by Mark Jaquith (<http://markjaquith.wordpress.com/2012/05/26/wordpress-skeleton/>) *[optional]*


Generally speaking, there are three categories of content which make a database-driven website tick:

 1. "Code"
    **Contains:** themes, scripts, plugins, config files etc...
    **Location:** FTP
    **Traditional GUI Management tool:** FTP program
    **Command-line Management tool:** git
 
 2. "Media"
    **Contains:** whatever is stored in your uploads directory: images, audio, video, pdf's etc...
    **Location:** FTP
    **Traditional GUI Management tool:** FTP program
    **Command-line Management tool:** rsync

 3. "SQL data" 
    **Contains:** whatever is stored in your database: posts, pages, options, users, etc...
    **Location:** database
    **GUI Management tool:** PHPMyAdmin
    **Command-line Management tool:** syncdb (this script)

This script handles the migration of content categories 2) and 3). Part 1), the code, is managed in a Git repository on the remote server (see http://joemaller.com/990/a-web-focused-git-workflow/ for details). So finally, complete migration process looks like this:

**Pushing**:
    ./syncdb
    git push hub master


**Pulling**:
    ./syncdb pull
    git pull hub/master


Contribution
============

SyncDB has been tested on my machine paired with various shared web hosts, ranging from the good (Dreamhost), the bad (HostGator), and the ugly (GoDaddy).

In other words, this script has not seen a broad range of use cases. It presently only works with Wordpress, but in principle it should work with Drupal, Joomla, etc... Right now it scours the wp-config.php file for database login credentials (user, pass, server-name, host). All that would be required to support those other CMS's would be to change the config file and adjust sed's grep command accordingly.

Bug reports, suggestions, and pull-requests are welcome.


