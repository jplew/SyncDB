SyncDB v0.1
==========

SyncDB is bash deploy script meant to take the tedium out of synchronizing local and remote versions of a Wordpress site. It allows developers working in a local environment (eg. MAMP) to rapidly "push" or "pull" changes to or from their production server with a single terminal command. 

SyncDB synchronizes your sites by executing a sequence of shell commands. It uses `mysql` and `mysqldump` for your database, `rsync` for your uploads folder, and leaves it up to you to synchronize your themes, plugins, etc (I use `git` for this).

Search and replacing the site URLs in the database is handled by the CLI version of InterConnect IT's invaluable [Wordpress Search and Replace Tool](http://interconnectit.com/products/search-and-replace-for-wordpress-databases). 

After initial script configuration, you'll be able to migrate your entire Wordpress database and uploads folder by running SyncDB from the command line like so:

    ./syncdb

No more FTP programs, no more PHPMyAdmin, no more tears. SyncDB don't mess around.



## Usage

1. Copy syncdb to the root of your site and make sure it's executable (`chmod +x syncdb`).
    + This is important, as the script must be located in the same folder as your wp-config.php (and your local-config.php, if you're doing it the smart way ([Mark Jaquith's WP-Skeleton](http://markjaquith.wordpress.com/2012/05/26/wordpress-skeleton/)).
2. Configure the script
    + This step involves manually editing the variables located at the top of the script using your favorite text editor. This has to be done once for each local/remote site pair. All the settings are documented via comments in the script itself.

3. Run syncdb command from the command line: `./syncdb [command]`

SyncDB works in either direction. It can "push" your changes to the remote server, or "pull" updates from it.

To **push**, run:

    ./syncdb

or

    ./syncdb push

To **pull**, run:

    ./syncdb pull

For finer control, you can tell SyncDB to execute only the command of your choosing by passing the function name as an argument. See **List of Commands** below.




## List of Commands


Below is a list of all of SyncDB's commands. They are meant to be passed one at a time like so:

    ./syncdb backup_local_db
  
All commands are meant to be executed locally, unless noted with *(remote only)*. 

   + push *(default)*
     + Execute the following sequence of commands:
         + `test_ssh()`
         + `backup_local_db()`
         + `upload_local_db()`
         + `upload_script()`
         + `do_remote_backup()`
         + `do_remote_operations()`
         + `do_search_replace_remote()`
         + `rsync_push()`

   + pull
     + Execute the following sequence of commands:
         + `test_ssh()`
         + `upload_script()`
         + `do_remote_backup()`
         + `download_remote_db()`
         + `backup_local_db()`
         + `replace_local_db()`
         + `search_replace_local()`
         + `rsync_pull()`

   + test_ssh
     + Check if the SSH connection is working.

   + backup\_local\_db
     + Back up the local MySQL database to latest-local.mssql.bz2 (or whatever you set $l\_db\_name to). Another backup dump is created with its name prepended by the current date and time. For example 20130902-1230-database.mssql.bz2.

   + upload\_local\_db
     + Upload the most recent local database dumpfile to the remote server.

   + upload\_script
     + Upload a copy of SyncDB to your remote host's root directory.

   + do\_remote\_backup
     + Login to the remote server via SSH and run `backup\_remote\_db()`, which will backup the remote MySQL database.

   + backup\_remote\_db
     + *(remote only)* Backup the remote MySQL database to latest-remote.mssql.bz2 (or whatever you set $r\_db\_name to). Another backup dump is created with its name prepended by the current date and time. For example 20130902-1230-database.mssql.bz2.

   + do\_search\_replace\_remote
     + Login to remote server and execute `search\_replace\_remote()`.

   + download\_remote\_db
     + Download the most recent remote database dump file via `scp`.

   + replace\_local\_db
     + Drop then recreate the local database. This effectively deletes all the tables.

   + do\_remote\_operations
     + Login to remote server and execute `replace\_remote\_db()` method.

   + replace\_remote\_db
     + *(remote only)* Drop and recreate remote database. This effectively deletes all the tables.

   + search\_replace\_local
     + Download the Search and Replace tools locally then execute them.

   + search\_replace\_remote
     + *(remote only)* Download the Search and Replace tools on the remote server, execute them, then delete them.
 
   + rsync\_pull
     + Synchronize the local uploads folder with the contents of your remote uploads.

   + rsync\_push
     + Synchronize the remote uploads folder with the contents of your local uploads 


## Complete workflow


I wrote this script due to my recurring need to migrate Wordpress sites. I required a workflow which would automate the process as much as possible, with minimal intervention. This script, therefore, plays one part in my overall Wordpress development workflow. Below is my set-up:

### Local

 * Mac OS X 10.8.4 running [MAMP](<http://www.mamp.info>) *[optional]*
 * Name-based virtual hosting enabled (eg. http://dev.mywebsite.com as opposed to localhost/mywebsite) *[recommended]*

### Remote

 * SSH enabled *[required]*
 * Password-less login with SSH Keys enabled ([a guide](<http://sniptools.com/mac-osx/save-ssh-password-in-terminal>)) *[recommended]*
  
### Common

 * Git-managed code using Joe Maller's awesome [Web-focused Git workflow](<http://joemaller.com/990/a-web-focused-git-workflow/>) *[optional]*
 * Wordpress installation using Mark Jaquith's [WP-Skeleton](<http://markjaquith.wordpress.com/2012/05/26/wordpress-skeleton/>) *[recommended]*
     + one key benefit is that you require only one version of `wp-config.php` between your local and remote host. Therefore, you don't have to modify it each time you push or pull changes. Make sure your `local-config.php` has been added to your `.gitignore`. 

### The Model

This workflow is based on the following model. Generally speaking, there are three categories of content which make a database-driven website tick:

 1. **"Code"**
    + **Contains:** themes, scripts, plugins, config files etc...
    + **Location:** FTP
    + **Traditional GUI Management tool:** FTP program
    + **Command-line Management tool:** `git`

 2. **"Media"**
    + **Contains:** whatever is stored in your uploads directory: images, audio, video, docs etc...
    + **Location:** FTP
    + **Traditional GUI Management tool:** FTP program
    + **Command-line Management tool:** `rsync`

 3. **"SQL data"**
    + **Contains:** whatever is stored in your database: posts, pages, options, users, etc...
    + **Location:** MySQL database
    + **GUI Management tool:** PHPMyAdmin
    + **Command-line Management tool:** `syncdb` (this script)

This script handles the migration of content categories 2) and 3). Part 1), the code, is managed in a Git repository on the remote server. Once this workflow has been enabled, the complete migration process looks like this:

 **Pushing**:
 
     ./syncdb
     git push hub master
 
 
 **Pulling**:
 
     ./syncdb pull
     git pull hub/master

Easy!


## Feedback & Bugs


SyncDB may be a little edgy—it has not seen a broad range of use cases. I have used it to good effect between my machine and numerous shared hosting plans, ranging from the good (Dreamhost), the bad (HostGator), and the ugly (GoDaddy). 

It is presently compatible with Wordpress, but in principle it should work with Drupal, Joomla, etc... Right now it scrapes the wp-config.php file for database login credentials. All that would be required to support those other CMS's would be to modify the config file and the `sed` command accordingly.

Bug reports, suggestions, and pull-requests are welcome.
