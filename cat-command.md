# To overwrite the file:
```
cat <<EOL > $WORKDIR_LOCUST/.env
LOCUST_WEB_LOGIN=$LOCUST_WEB_LOGIN
FLASK_SECRET_KEY=$FLASK_SECRET_KEY
LOCUST_USERNAME=$LOCUST_USERNAME
LOCUST_PASSWORD=$LOCUST_PASSWORD
EOL
```
# To append to the file, newline:
```
cat <<EOL >> $WORKDIR_LOCUST/.env
LOCUST_WEB_LOGIN=$LOCUST_WEB_LOGIN
FLASK_SECRET_KEY=$FLASK_SECRET_KEY
LOCUST_USERNAME=$LOCUST_USERNAME
LOCUST_PASSWORD=$LOCUST_PASSWORD
EOL
```

#
# This will replace the line starting with LOCUST_WEB_LOGIN= with the new value without overwriting the entire file.
```
sed -i 's/^LOCUST_WEB_LOGIN=.*/LOCUST_WEB_LOGIN_NEW_STRING=$LOCUST_WEB_LOGIN_NEW_STRING/' $WORKDIR_LOCUST/.env
```


# Clear history in linux
```
cat /dev/null > ~/.bash_history && history -c && exit
```
