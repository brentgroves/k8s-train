https://www.loginradius.com/blog/engineering/rest-api-kubernetes/

apiVersion: v1

kind: ConfigMap

metadata:

  name: app-config

data:

  FLASK_APP: app.py

  MYSQL_ROOT_USER: root

  MYSQL_ROOT_PASSWORD: admin

  MYSQL_ROOT_HOST: "<plase holder>"

  MYSQL_ROOT_PORT: "3306"

  MYSQL_ROOT_DB: mydb

  FLASK_APP_PORT: "8282"