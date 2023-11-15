# Backup MySQL Database on Shared Dreamhost instance
Inspired from this tutorial by Nick Janetakis: [Automatic MySQL / PostgreSQL Backups with a Shell Script and Cron Job](https://www.youtube.com/watch?v=kbCytSYPh0E)\
Code inspired from [@nickjj/cron-sendy-backup](https://gist.github.com/nickjj/00b07e522caee02e37951ec6de2a9c95)

These scripts may also work on other webhost providers not just DreamHost. Just change the `config_path` and the way cron is setup.

**This script will split the encrypted mysql dump file into by 20 megabytes for e-mail attachment.**

**This branch processes CodeIgniter 3 `database.php` file.**\
For the WordPress version, checkout the [master](https://github.com/TechnologyXING/mysql-backup-dreamhost/tree/master) branch.\
For the .env version, checkout the [env](https://github.com/TechnologyXING/mysql-backup-dreamhost/tree/env) branch.

## Setup
### Server Requirements
`mutt` should be installed on the server and accessible via commandline. See: [mutt](http://www.mutt.org/)

Create a `.muttrc` in your home folder, ex. `~/.muttrc`, that contains your SMTP settings. Modify the below configuration for your setup.
```
# About Me
set from = "sender@email.com"
set realname = "Live Backups"

# My credentials
set smtp_url = "smtp://sender@email.com@email.com:587"
set smtp_pass = "smpt_password"
set imap_user = "sender@email.com"
set imap_pass = "imap_password"

# My mailboxes
set folder = "imaps://imap.email.com:993"
set spoolfile = "+INBOX"


# Where to put the stuff
set header_cache = "~/.mutt/cache/headers"
set message_cachedir = "~/.mutt/cache/bodies"
set certificate_file = "~/.mutt/certificates"

# Etc
set mail_check = 30
set move = no
set imap_keepalive = 900
set sort = threads
set editor = "vim"

# GnuPG bootstrap
# source ~/.mutt/gpg.rc
```

### Instructions
[Download](https://github.com/TechnologyXING/mysql-backup-dreamhost/archive/ci3-split.zip) and unzip the code in your local folder.\
*Direct checkout of this repo to your server is NOT ADVISABLE as configs may change.*

Add a new configuration in your CodeIgniter 3 `database.php` file where the database backup will be sent.
```
$db['default']['notifyme'] = "admin@example.com"
```

Modify the contents of `.my.cnf` file with a username and password that has access to the target database. [*](https://stackoverflow.com/questions/9293042/how-to-perform-a-mysqldump-without-a-password-prompt) \
This user may be a different user from the target `database.php` as long as that user has dump access on the target database and access to **information_schema**.

Change the access of the `.my.conf` file to `600` or `400`
```
$ chmod 600 .my.cnf
```

Modify the `config_path` variable in the `mysql-backup` and `mysql-restore` files.
```
config_path="/home/techxing/techxing.net/application/config"
# enter the absolute path to the directory where the database.php is located. do not include database.php and closing slash
```

Modify the `/absolute/path/to/mysql-backup-dreamhost` in the `cron-mysql-backup` file. *(Optional file included here as reference for a different cron setup. Copy the contents of this file to your `crontab -e` to run the `mysql-backup` hourly.)* 

Modify the `/absolute/path/to/mysql-backup-dreamhost` in the `cron-mysql-backup-rotate` file. *(Optional file included here as reference for a different cron setup. Copy the contents of this file to your `crontab -e` to delete messages older than 90 days. Runs once a day)*
You can increase or decrease the retention period of the files by modifying the `+90` (*checks if older than 90 days*) variable.

Upload the `mysql-backup-dreamhost` folder to your DreamHost Shared server in the `/home/<dreamhost user>/` folder via FTP, SSH, or SCP etc. creating the following directory structure (example):
 ```
/home/techxing/mysql-backup-dreamhost
 ```
> **Make sure that `mysql-backup-dreamhost` is not publicly accessible**

You may need to add execute permissions to the files:
```
$ chmod +x mysql-backup
$ chmod +x mysql-restore
```

With this defaut setup, the script will access the `/home/techxing/techxing.net/application/config/database.php` file.\
And this will create the backups in `/home/techxing/mysql-backup-dreamhost/backups/` folder.

> **Tip**: Upload the `mysql-backup-dreamhost` folder with a new name and modify the config to create a new instance of the backup. Then create a different cron job for the new instance.

On your server, `cd` to the uploaded `mysql-backup-dreamhost` create a new encryption keys for the mysqldump.[*](https://gist.github.com/pugsley/2914b9c0d1e7b866eab2a4cc0ceb0ead)

```
$ openssl req -x509 -nodes -newkey rsa:2048 -keyout mysqldump-key.priv.pem -out mysqldump-key.pub.pem
```

To combine the split parts of the dump:
```
$ cat [db_name]_[archive_date]-part_* >> [filename].sql.gz.enc
```

To decrypt and decompress to backups:
```
$ openssl smime -decrypt -in [filename].sql.gz.enc -binary -inform DEM -inkey mysqldump-key.priv.pem -out [filename].sql.gz
$ gzip -d [filename].sql.gz
```


## Testing
This requires that the DreamHost user must have SSH/Secure shell access enabled: [Editing an existing user to become a SHELL user](https://help.dreamhost.com/hc/en-us/articles/216385837-Creating-a-user-with-Shell-SSH-access)

To test the variables that are extracted from the file uncomment this line before the `mysqldump` command and execute the file in the command line to display the values: (example) `./mysql-backup` or `./mysql-restore`
```
printf "VARS: \n\${db_conf} = '${db_conf}' \n\${db_name} = '${db_name}' \nDESTINATION: '${backup_destination}/${db_name}_$(date +%Y%m%d-%H%M%S).sql.gz' \n"; exit;
```
*Values between the single quotes `'` are the actual values*

## Creating a cron job in DreamHost
[Source](https://help.dreamhost.com/hc/en-us/articles/215088668-How-do-I-create-a-cron-job-)

**Pro Tip**: Test your commands first in the command line (without the delete option) before entering them in this cron job.

### Cron Job (Backup)
Go to your [Cron Jobs](https://panel.dreamhost.com/index.cgi?tree=advanced.cron&) panel.

Add a new Cron Job.

Select the DreamHost user from the [Setup](#setup) section.

Type in a meaningful title for the job ex. `Daily DB Backup of techxing.net`.

(Optional) Enter an email to notify.

Enter the full path of the `mysql-backup` in the command to run, (example)
```
/home/techxing/mysql-backup-dreamhost/mysql-backup
```

Set to run `Daily`

Submit the form.

### Cron Job (Delete old archives)
Go to your [Cron Jobs](https://panel.dreamhost.com/index.cgi?tree=advanced.cron&) panel.

Add a new Cron Job.

Select the DreamHost user from the [Setup](#setup) section.

Type in a meaningful title for the job (example) `Cleanup DB Backup more than 90 days of techxing.net`.

(Optional) Enter an email to notify.

Enter the following in the command to run, (example)
```
find /home/techxing/mysql-backup-dreamhost/backups -type f -mtime +90 -delete > /dev/null 2>&1
```
*Where `/home/techxing/mysql-backup-dreamhost/backups` is the absolute path to the backups folder*

Set to run `Daily`

Submit the form.

---

## Caveat
Script currently doesn't work as-is with database credentials with a blank password (eg. localhost).\
A work around for this is to set the `db_pass` variable manually to '' (blank, empty). Example:
```
db_pass="" #"$(grep -o "DB_PASSWORD=[^']*[^']" "${db_conf}" | cut -d "=" -f 2 | sed "s/'//g;s/ //g")"
```
