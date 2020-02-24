# invincible-env

![Architecture Diagram](architecture-diagram.png?raw=true)

Sets up invincible ecosystem on Azure.  After this script is done, you will need to integrate the Azure CI/CD with the invincible webapp.  This can be done via a command similar to the following:

```
az webapp deployment source config \
    -g $resourceGroupName \
    -n $appName \
    -u https://github.com/mruptown/invincible \
    --branch master \
    --git-token a1b2c3d4e5f6g7h8i9j10k11l12
```

... where you substitute out your project and your git-token

An entry has been put in the .gitignore file called `deployapp.azcli` if you'd like to store the above command in there.
