#Linux Server Config
Recommend using ubuntu documentation instead of any website tutorials to ensure that

http://www.pathname.com/fhs/pub/fhs-2.3.pdf
##Create new user
additional info: https://help.ubuntu.com/community/AddUsersHowto
First thing we want to do is create new user with sudo access so we can disable remote
login of root user.  We do this by entering the command
```
sudo adduser grader
```
where "grader" is the name of the user you want to create.
###Give user ssh access

