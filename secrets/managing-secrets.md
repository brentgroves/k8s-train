https://kubernetes.io/docs/tasks/configmap-secret/managing-secret-using-kubectl/

Create a Secret
A Secret can contain user credentials required by pods to access a database. For example, a database connection string consists of a username and password. You can store the username in a file ./username.txt and the password in a file ./password.txt on your local machine.

    username = 'mg.odbcalbion' 
    # password = 'wrong' 
    password = 'Mob3xalbion' 

This seemed to work best.
echo "mg.odbcalbion" > ./username.txt
echo "Mob3xalbion" > ./password.txt
echo "mgadmin" > ./username2.txt
echo "WeDontSharePasswords1\!" > ./password2.txt

This does not work because a % character is added to the file.
printf %s "mg.odbcalbion" >> ./username.txt
printf %s "Mob3xalbion" >> ./password.txt

This does not work because a % character is added to the file.
echo -n 'mg.odbcalbion' > ./username.txt
echo -n 'Mob3xalbion' > ./password.txt

In these commands, the -n flag ensures that the generated files do not have an extra newline character at the end of the text. This is important because when kubectl reads a file and encodes the content into a base64 string, the extra newline character gets encoded too.

The kubectl create secret command packages these files into a Secret and creates the object on the API server.

kubectl create secret generic db-user-pass \
  --from-file=username=./username.txt \
  --from-file=password=./password.txt \
  --from-file=username2=./username2.txt \
  --from-file=password2=./password2.txt \
  --from-file=username3=./username3.txt \
  --from-file=password3=./password3.txt

kubectl create secret generic db-user-pass \
  --from-literal=username='mg.odbcalbion' \
  --from-literal=password='Mob3xalbion'
  --from-literal=username2='mgadmin' \
  --from-literal=password2='WeDontSharePasswords1!'

kubectl delete secret db-user-pass

kubectl get secret db-user-pass -o jsonpath='{.data}'
kubectl get secret db-user-pass -o jsonpath='{.data.password}' | base64 --decode

kubectl delete secret db-user-pass

kubectl get secret db-user-pass -o jsonpath='{.data}'
kubectl get secret db-user-pass -o jsonpath='{.data.password}' | base64 --decode
kubectl create secret generic db-user-pass \
  --from-file=./username.txt \
  --from-file=./password.txt

  The output is similar to:

secret/db-user-pass created
The default key name is the filename. You can optionally set the key name using --from-file=[key=]source. For example:

kubectl create secret generic db-user-pass \
  --from-file=username=./username.txt \
  --from-file=password=./password.txt
You do not need to escape special characters in password strings that you include in a file.

You can also provide Secret data using the --from-literal=<key>=<value> tag. This tag can be specified more than once to provide multiple key-value pairs. Note that special characters such as $, \, *, =, and ! will be interpreted by your shell and require escaping.

In most shells, the easiest way to escape the password is to surround it with single quotes ('). For example, if your password is S!B\*d$zDsb=, run the following command:

kubectl create secret generic db-user-pass \
  --from-literal=username=devuser \
  --from-literal=password='S!B\*d$zDsb='
Verify the Secret
Check that the Secret was created:

kubectl get secrets

The output is similar to:

NAME                  TYPE                                  DATA      AGE
db-user-pass          Opaque                                2         51s
You can view a description of the Secret:

kubectl describe secrets/db-user-pass
The output is similar to:

Name:            db-user-pass
Namespace:       default
Labels:          <none>
Annotations:     <none>

Type:            Opaque

Data
====
password:    12 bytes
username:    5 bytes
The commands kubectl get and kubectl describe avoid showing the contents of a Secret by default. This is to protect the Secret from being exposed accidentally, or from being stored in a terminal log.

Decoding the Secret 
To view the contents of the Secret you created, run the following command:

kubectl get secret db-user-pass -o jsonpath='{.data}'
The output is similar to:

{"password":"MWYyZDFlMmU2N2Rm","username":"YWRtaW4="}

Now you can decode the password data:

# This is an example for documentation purposes.
# If you did things this way, the data 'MWYyZDFlMmU2N2Rm' could be stored in
# your shell history.
# Someone with access to you computer could find that remembered command
# and base-64 decode the secret, perhaps without your knowledge.
# It's usually better to combine the steps, as shown later in the page.
echo 'MWYyZDFlMmU2N2Rm' | base64 --decode
The output is similar to:

1f2d1e2e67df
In order to avoid storing a secret encoded value in your shell history, you can run the following command:

kubectl get secret db-user-pass -o jsonpath='{.data.password}' | base64 --decode
The output shall be similar as above.


