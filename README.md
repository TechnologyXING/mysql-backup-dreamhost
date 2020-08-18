# Backup MySQL Database on Shared Dreamhost instance
Inspired from [@nickjj/cron-sendy-backup](https://gist.github.com/nickjj/00b07e522caee02e37951ec6de2a9c95)
Tutorial: [Automatic MySQL / PostgreSQL Backups with a Shell Script and Cron Job](https://www.youtube.com/watch?v=kbCytSYPh0E)

## Requirement


## Setup
Upload the ```mysql-backup-dreamhost``` folder to your DreamHost Shared server in the ```/home/<dreamhost user>/``` folder via FTP, SSH, or SCP etc. creating the following directory structure:
 ```
/home/techxing/mysql-backup-dreamhost
 ```
*Make sure that ```mysql-backup-dreamhost``` is not publicly accessible*

Modify the following variables in the  ```mysql-backup``` and ```mysql-restore``` files.
```
dh_user='techxing' # enter your DreamHost SSH username here
dh_web_directory='techxing.net' # enter the your DreamHost web directory here
```
With this setting, the script will access the ```wp-config.php``` in ```/home/techxing/techxing.net/wp-config.php```.
And will create the backups in ```/home/techxing/mysql-backup-dreamhost/backups/<wp database>/``` folder.

Modify the ```<dreamhost user>``` in the ```cron-mysql-backup``` file.

Modify the ```<dreamhost user>``` in the ```cron-mysql-backup-rotate``` file.
You can increase or decrease the retention period of the files by modifying the ```+90``` (90 days) variable.

## Testing
This requires that the DreamHost user must have SSH/Secure shell access enabled: [Editing an existing user to become a SHELL user](https://help.dreamhost.com/hc/en-us/articles/216385837-Creating-a-user-with-Shell-SSH-access)

To test the variables that are extracted from the file ucomment this line before the ```mysqldump``` command and execute the file in the command line to display the values: ex. ```./mysql-backup``` or ```./mysql-restore```
```
printf "VARS: \n\${db_conf} = '${db_conf}' \n\${db_name} = '${db_name}' \n\${db_user} = '${db_user}' \n\${db_pass} = '${db_pass}' \n\${db_host} = '${db_host}' \nDESTINATION: '${backup_destination}/${db_name}_$(date +%F).sql.gz' \n"; exit;
```
*Values between the single quotes ```'``` are the actual values*
