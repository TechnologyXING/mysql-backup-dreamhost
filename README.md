# Backup MySQL Database on Shared Dreamhost instance
Inspired from [@nickjj/cron-sendy-backup](https://gist.github.com/nickjj/00b07e522caee02e37951ec6de2a9c95)\
Tutorial: [Automatic MySQL / PostgreSQL Backups with a Shell Script and Cron Job](https://www.youtube.com/watch?v=kbCytSYPh0E)

## Setup
Upload the ```mysql-backup-dreamhost``` folder to your DreamHost Shared server in the ```/home/<dreamhost user>/``` folder via FTP, SSH, or SCP etc. creating the following directory structure:
 ```
/home/techxing/mysql-backup-dreamhost
 ```
*Make sure that ```mysql-backup-dreamhost``` is not publicly accessible*

You may need to add execute permissions to the files:
```
$ chmod +x mysql-backup
$ chmod +x mysql-restore
```

Modify the following variables in the  ```mysql-backup``` and ```mysql-restore``` files.
```
dh_user='techxing' # enter your DreamHost SSH username here
dh_web_directory='techxing.net' # enter the your DreamHost web directory here
```
With this setting, the script will access the ```wp-config.php``` in ```/home/techxing/techxing.net/wp-config.php```.\
And will create the backups in ```/home/techxing/mysql-backup-dreamhost/backups/<wp database>/``` folder.

*(Optional file left here as reference)* Modify the ```<dreamhost user>``` in the ```cron-mysql-backup``` file.

*(Optional file left here as reference)* Modify the ```<dreamhost user>``` in the ```cron-mysql-backup-rotate``` file.\
You can increase or decrease the retention period of the files by modifying the ```+90``` (90 days) variable.

## Testing
This requires that the DreamHost user must have SSH/Secure shell access enabled: [Editing an existing user to become a SHELL user](https://help.dreamhost.com/hc/en-us/articles/216385837-Creating-a-user-with-Shell-SSH-access)

To test the variables that are extracted from the file ucomment this line before the ```mysqldump``` command and execute the file in the command line to display the values: ex. ```./mysql-backup``` or ```./mysql-restore```
```
printf "VARS: \n\${db_conf} = '${db_conf}' \n\${db_name} = '${db_name}' \n\${db_user} = '${db_user}' \n\${db_pass} = '${db_pass}' \n\${db_host} = '${db_host}' \nDESTINATION: '${backup_destination}/${db_name}_$(date +%Y%m%d-%H%M%S).sql.gz' \n"; exit;
```
*Values between the single quotes ```'``` are the actual values*

## Caveat
Script currently doesn't work as-is with database user with a blank password (eg. localhost).\
A work around for this is to set the ```db_pass``` variable manually to ''. ex:
```
db_pass='' #"$(grep -o "define('DB_PASSWORD', '[^']*[^');]'" "${db_conf}" | cut -d "," -f 2 | sed "s/'//g;s/ //g")"
```

## Creating a cron job in DreamHost
[Source](https://help.dreamhost.com/hc/en-us/articles/215088668-How-do-I-create-a-cron-job-)

Pro Tip: Test your commands first in the command line before entering them in this cros job.

### Cron Job (Backup)
Go to your [Cron Jobs](https://panel.dreamhost.com/index.cgi?tree=advanced.cron&) panel.

Add a new Cron Job.

Select the DreamHost user from the [Setup](#setup) section.

Type in a meaningful title for the job ex. ```Daily DB Backup of techxing.net```.

(Optional) Enter an email to notify.

Enter the full path of the ```mysql-backup``` in the command to run ex.
```
/home/techxing/mysql-backup-dreamhost/mysql-backup
```

Set to run ```Daily```

Submit the form.

### Cron Job (Delete old archives)
Go to your [Cron Jobs](https://panel.dreamhost.com/index.cgi?tree=advanced.cron&) panel.

Add a new Cron Job.

Select the DreamHost user from the [Setup](#setup) section.

Type in a meaningful title for the job ex. ```Cleanup DB Backup more than 90 days of techxing.net```.

(Optional) Enter an email to notify.

Enter the full path of the ```mysql-backup``` in the command to run ex.
```
find /home/techxing/mysql-backup-dreamhost/backups -type f -mtime +90 -delete > /dev/null 2>&1
```

Set to run ```Daily```

Submit the form.