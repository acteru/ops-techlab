Lab 3.3: Daily business
============

Lab 3.3.1: User management
-------------
## Add user to project
First we create a user and give him the admin role in the openshift-infra project.
Login to the master and create the local user
```
[ec2-user@master0 ~]$ sudo htpasswd /etc/origin/master/htpasswd cowboy
```

Add the admin role of the project openshift-infra to the created user.
```
[ec2-user@master0 ~]$ oc adm policy add-role-to-user admin cowboy -n openshift-infra
```

Now login with the new user from your client and check if you see the openshift-infra project
```
[localuser@localhost ~]$ oc login https://console.user[X].lab.openshift.ch
Username: cowboy
Password:
Login successful.

You have one project on this server: "openshift-infra"

Using project "openshift-infra".
```

## Add cluster role to user
If the user is a cluster admin we should delete the created rolebinding for the project and give him the global clusterPolicyBinding "cluster-admin" role.

Login as sheriff
```
[ec2-user@master0 ~]$ oc login -u sheriff
```

Add the cluster-admin role to the created user.
```
[ec2-user@master0 ~]$ oc adm policy remove-role-from-user admin cowboy -n openshift-infra
role "admin" removed: "cowboy"
[ec2-user@master0 ~]$ oc adm policy add-cluster-role-to-user cluster-admin cowboy
cluster role "cluster-admin" added: "cowboy"
```
Now you can try to login from your client with the user and check if you see all projects
```
[localuser@localhost ~]$ oc login https://console.user[X].lab.openshift.ch
Authentication required for https://console.user[X].lab.openshift.ch (openshift)
Username: cowboy
Password:
Login successful.

You have access to the following projects and can switch between them with 'oc project <projectname>':

    appuio-infra
    default
    kube-public
    kube-system
    logging
    management-infra
    openshift
  * openshift-infra

Using project "openshift-infra".
```

## Create a group and add user to the group
It is also possible to add a group with users to a cluster policy.
Create a local group and add cowboy to it. (It's also possible to work with ldap/AD groups)
```
[ec2-user@master0 ~]$ oc adm groups new test-group cowboy
NAME         USERS
test-group   cowboy
```

Add the group to the cluster-role
```
[ec2-user@master0 ~]$ oc adm policy add-cluster-role-to-group cluster-admin test-group
cluster role "cluster-admin" added: "test-group"
```

Verifiy that the group is added to the cluster-admins
```
[ec2-user@master0 ~]$ oc get clusterrolebindings | grep cluster-admin
cluster-admin                                                         /cluster-admin                                                         sheriff, cowboy                system:masters, test-group               
```

## Evaluate authorizations
It's possible to evaluate authorizations. This can be done with the following pattern.
```
oc policy who-can VERB RESOURCE_NAME
```

Examples
Who can delete the openshift-infra project.
```
oc policy who-can delete project -n openshift-infra
```

Who can create configmaps in the default project.
```
oc policy who-can create configmaps -n default
```

## Delete resource
Delete the user and the group:
```
[ec2-user@master0 ~]$ oc delete group test-group
[ec2-user@master0 ~]$ oc get identity
[ec2-user@master0 ~]$ oc delete identity htpasswd_auth:cowboy
[ec2-user@master0 ~]$ sudo htpasswd -D /etc/origin/master/htpasswd cowboy
```

You can get all available clusterPolicies and clusterPoliciesBinding with the following oc command.
```
[ec2-user@master0 ~]$ oc login -u sheriff

[ec2-user@master0 ~]$ oc describe clusterPolicy default
[ec2-user@master0 ~]$ oc describe clusterPolicyBindings :default
```

---

**End of Lab 3.3.1**

<p width="100px" align="right"><a href="332_update_host.md">Update Host →</a></p>

[← back to overview](../README.md)
