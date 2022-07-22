
## "Getting Started"
We build upon Kubeflow Documentation on multiuser isolation provided at: https://www.kubeflow.org/docs/components/multi-tenancy/getting-started/ and highlight how to add new users to Kubeflow.

## Create user profile

Create a `profile.yaml` file with the following content on your local machine:
```
apiVersion: kubeflow.org/v1beta1
kind: Profile
metadata:
  name: profileName   # replace with the name of profile you want, this will be user's namespace name
spec:
  owner:
    kind: User
    name: userid@email.com   # replace with the email of the user

  resourceQuotaSpec:    # resource quota can be set optionally
   hard:
     cpu: "2"
     memory: 2Gi
     persistentvolumeclaims: "1"
     requests.storage: "5Gi"
```
Run the following command to create resource:
```
kubectl create -f profile.yaml
```
Respose to this should be:
```
profile.kubeflow.org/newuser created
```

Kubeflow creates a new namespace with the name specified by `name` in the above yaml file.
```
$ kubectl get namespaces
```
will show a new namespace `newuser` created.

## Add user email and password

We have created a user namespace and specified his email with `name: userid@email.com` in the `profile.yaml`.
Lets add this user to our dex configmap. To add this user's password we rely on hashed password using bcrypt hash generator. There are multiple websites online through which you can generate a BCrypt hash. 

```
export EDITOR=nano
kubectl edit configmap dex -n auth -o yaml
```
and edit this configmap:
```
staticPasswords:
  - email: userid@email.com
    hash: dgdsgKgfgnk
    username: profileName
  - email: default login from kubrflow
    hash: default password hash used by Kubeflow
    username: default username of kubeflow
```

Rollout restart
```
kubectl rollout restart deployment dex -n auth
```

This responds in:
```
deployment.apps/dex restarted
```

Now again login to Kubeflow using newly created userid and password. 
