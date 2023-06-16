# Blog
[Ghost CMS](https://www.ghost.org/) blog


# Setup
1) set MYSQL_ROOT_PASSWORD as environment variable (e.g. from .env file)
2) run ```docker-compose up -d ```

# Backup
The backup consists of two sections:
1) MYSQL backup - databases mysql and ghost
```
    docker exec blog-db-1 /usr/bin/mysqldump -u root --password=$MYSQL_ROOT_PASSWORD --no-tablespaces ghost > /home/mike/blog/backup/backup_ghost.sql
    docker exec blog-db-1 /usr/bin/mysqldump -u root --password=$MYSQL_ROOT_PASSWORD --no-tablespaces mysql > /home/mike/blog/backup/backup_mysql.sql
```

2) Content backup
The folder content which is mounted as volume in the container needs to be backed up.
```
   /content:/var/lib/ghost/content
```

# Mount and copy to NAS
```
if ! cat /proc/mounts | grep nas; then 
    mount -t nfs [NAS IP]:/volume1/linux /mnt/nas
else 
    echo "already mounted"
fi
```

The backup files will then be transferred to my NAS.
```
cp -r /home/mike/blog/backup/backup* /mnt/nas/Blog 
cp -r /home/mike/blog/content/ /mnt/nas/Blog       
```

# Restore
1) Run docker-compose as in [Setup](#setup)
2) Drop volume ```docker volume rm blog_mysql-data```
3) Copy the backup files from the NAS to the current directory
   ```
    cp /mnt/nas/Blog/backup_ghost.sql .
    cp /mnt/nas/Blog/backup_mysql.sql .
    cp -r /mnt/nas/Blog/content . 
   ```
4) Restore mysql Tables 
    ```
        docker exec -i blog-db-1 mysql -u root --password=$MYSQL_ROOT_PASSWORD ghost < backup_ghost.sql
        docker exec -i blog-db-1 mysql -u root --password=$MYSQL_ROOT_PASSWORD mysql < backup_mysql.sql
    ```


