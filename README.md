# Backup MySQL Database on Shared Dreamhost instance
Inspired from this tutorial by Nick Janetakis: [Automatic MySQL / PostgreSQL Backups with a Shell Script and Cron Job](https://www.youtube.com/watch?v=kbCytSYPh0E)\
Code inspired from [@nickjj/cron-sendy-backup](https://gist.github.com/nickjj/00b07e522caee02e37951ec6de2a9c95)

These scripts may also work on other webhost providers not just DreamHost. Just change the ```config_path`` and the way cron is setup.

**This branch processes CodeIgniter 3 ```database.php``` files.**\
For the WordPress version, checkout the [master](https://github.com/TechnologyXING/mysql-backup-dreamhost/tree/master) branch.\
For the WordPress .env version, checkout the [master](https://github.com/TechnologyXING/mysql-backup-dreamhost/tree/env) branch.

## Setup
[Download](https://github.com/TechnologyXING/mysql-backup-dreamhost/archive/ci3.zip) and unzip the code in your local folder.\
*Direct checkout of this repo to your server is NOT ADVICED as configs may change.*

Modify the ```config_path``` variable in the  ```mysql-backup``` and ```mysql-restore``` files.
```
config_path="/home/techxing/techxing.net/application/config"
# enter the absolute path to the directory where the database.php is located. do not include database.php and closing slash
```

Modify the ```/absolute/path/to/mysql-backup-dreamhost``` in the ```cron-mysql-backup``` file. *(Optional file included here as reference for a different cron setup)*

Modify the ```/absolute/path/to/mysql-backup-dreamhost``` in the ```cron-mysql-backup-rotate``` file. *(Optional file included here as reference for a different cron setup)*
You can increase or decrease the retention period of the files by modifying the ```+90``` (*checks if older than 90 days*) variable.

Upload the ```mysql-backup-dreamhost``` folder to your DreamHost Shared server in the ```/home/<dreamhost user>/``` folder via FTP, SSH, or SCP etc. creating the following directory structure (example):
 ```
/home/techxing/mysql-backup-dreamhost
 ```
*Make sure that ```mysql-backup-dreamhost``` is not publicly accessible*

You may need to add execute permissions to the files:
```
$ chmod +x mysql-backup
$ chmod +x mysql-restore
```

With this setting, the script will access the ```/home/techxing/techxing.net/application/config/database.php``` file.\
And this will create the backups in ```/home/techxing/mysql-backup-dreamhost/backups/``` folder.

**Tip**: Upload the ```mysql-backup-dreamhost``` folder with a new name and modify the config to create a new instance of the backup. Then create a different cron job for the new instance.

## Testing
This requires that the DreamHost user must have SSH/Secure shell access enabled: [Editing an existing user to become a SHELL user](https://help.dreamhost.com/hc/en-us/articles/216385837-Creating-a-user-with-Shell-SSH-access)

To test the variables that are extracted from the file uncomment this line before the ```mysqldump``` command and execute the file in the command line to display the values: (example) ```./mysql-backup``` or ```./mysql-restore```
```
printf "VARS: \n\${db_conf} = '${db_conf}' \n\${db_name} = '${db_name}' \n\${db_user} = '${db_user}' \n\${db_pass} = '${db_pass}' \n\${db_host} = '${db_host}' \nDESTINATION: '${backup_destination}/${db_name}_$(date +%Y%m%d-%H%M%S).sql.gz' \n"; exit;
```
*Values between the single quotes ```'``` are the actual values*

## Creating a cron job in DreamHost
[Source](https://help.dreamhost.com/hc/en-us/articles/215088668-How-do-I-create-a-cron-job-)

**Pro Tip**: Test your commands first in the command line (without the delete option) before entering them in this cron job.

### Cron Job (Backup)
Go to your [Cron Jobs](https://panel.dreamhost.com/index.cgi?tree=advanced.cron&) panel.

Add a new Cron Job.

Select the DreamHost user from the [Setup](#setup) section.

Type in a meaningful title for the job ex. ```Daily DB Backup of techxing.net```.

(Optional) Enter an email to notify.

Enter the full path of the ```mysql-backup``` in the command to run, (example)
```
/home/techxing/mysql-backup-dreamhost/mysql-backup
```

Set to run ```Daily```

Submit the form.

### Cron Job (Delete old archives)
Go to your [Cron Jobs](https://panel.dreamhost.com/index.cgi?tree=advanced.cron&) panel.

Add a new Cron Job.

Select the DreamHost user from the [Setup](#setup) section.

Type in a meaningful title for the job (example) ```Cleanup DB Backup more than 90 days of techxing.net```.

(Optional) Enter an email to notify.

Enter the following in the command to run, (example)
```
find /home/techxing/mysql-backup-dreamhost/backups -type f -mtime +90 -delete > /dev/null 2>&1
```
*Where ```/home/techxing/mysql-backup-dreamhost/backups``` is the absolute path to the backups folder*

Set to run ```Daily```

Submit the form.

---

## Caveat
Script currently doesn't work as-is with database credentials with a blank password (eg. localhost).\
A work around for this is to set the ```db_pass``` variable manually to '' (blank, empty). Example:
```
db_pass="" #"$(grep -o "DB_PASSWORD=[^']*[^']" "${db_conf}" | cut -d "=" -f 2 | sed "s/'//g;s/ //g")"
```
