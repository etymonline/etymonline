# Etymonline

## Construction Sequence

1. 

    ```
    export LEGACY_DB=`docker run -d -v /var/lib/mysql etymonline/etymonline-legacy-db`
    export LEGACY_DB_NAME=`docker inspect -f "{{ .Name }}" $LEGACY_DB`
    ```

2. 

    ```
    export LEGACY_SITE=`docker run -d --link $LEGACY_DB_NAME:legacy_db etymonline/etymonline-legacy-site`
    export LEGACY_SITE_NAME=`docker inspect -f "{{ .Name }}" $LEGACY_SITE`
    ```

5. 

    ```
    export NGINX=`docker run -d -P --link $LEGACY_SITE_NAME:legacy_site etymonline/etymonline-nginx`
    export NGINX_NAME=`docker inspect -f "{{ .Name }}" $NGINX`
    ```

## Tidbits

* Import Environmental Variables

    ```
    source ~/.etymenv
    ```

* Pull Updates

    ```
    docker pull etymonline/etymonline-legacy-db
    docker pull etymonline/etymonline-legacy-site
    docker pull etymonline/etymonline-nginx
    ```

* Ephemeral Legacy Database Client

    ```
    docker run -i -t --rm --link $LEGACY_DB_NAME:db ubuntu bash
    ```

* Legacy Database Upgrade

    ```
    docker stop $NGINX
    docker rm $NGINX
    ```

    ```
    docker stop $LEGACY_SITE
    docker rm $LEGACY_SITE
    ```

    ```
    export LEGACY_DB_OLD=$LEGACY_DB
    
    docker stop $LEGACY_DB_OLD
    export LEGACY_DB=`docker run -d --volumes-from $LEGACY_DB_OLD etymonline/etymonline-legacy-db`
    export LEGACY_DB_NAME=`docker inspect -f "{{ .Name }}" $LEGACY_DB`
    docker rm $LEGACY_DB_OLD

    unset LEGACY_DB_OLD
    ```

    ```
    export LEGACY_SITE=`docker run -d --link $LEGACY_DB_NAME:legacy_db etymonline/etymonline-legacy-site`
    export LEGACY_SITE_NAME=`docker inspect -f "{{ .Name }}" $LEGACY_SITE`
    ```

    ```
    export NGINX=`docker run -d -p 80:80 --link $LEGACY_SITE_NAME:legacy_site etymonline/etymonline-nginx`
    export NGINX_NAME=`docker inspect -f "{{ .Name }}" $NGINX`
    ```

* Legacy Database Permissions Setup

    ```
    docker stop $LEGACY_DB
    docker run -i -t --rm --volumes-from $LEGACY_DB etymonline/etymonline-legacy-db bash
      /usr/sbin/mysqld &
      mysql -e "GRANT ALL ON *.* TO root@'%' IDENTIFIED BY '' WITH GRANT OPTION"
      mysqladmin shutdown
    docker start $LEGACY_DB
    ```

* Export Environmental Variables

    ```
    echo "export LEGACY_DB=$LEGACY_DB" > ~/.etymenv
    echo "export LEGACY_DB_NAME=$LEGACY_DB_NAME" >> ~/.etymenv

    echo "export LEGACY_SITE=$LEGACY_SITE" >> ~/.etymenv
    echo "export LEGACY_SITE_NAME=$LEGACY_SITE_NAME" >> ~/.etymenv

    echo "export NGINX=$NGINX" >> ~/.etymenv
    echo "export NGINX_NAME=$NGINX_NAME" >> ~/.etymenv
    ```
