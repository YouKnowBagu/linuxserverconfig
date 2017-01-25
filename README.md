# Linux Server Config

This project will guide you through setting up a new Linux (Ubuntu) instance to serve an item catalog web application. The item catalog for my project can be found at <http://35.164.254.61/>, but will be taken offline upon completion of the Nanodegree.

# Resources

In addition to the usual Google and Stack Overflow, I also used the [Ubuntu community built wiki](https://help.ubuntu.com/community/) and the [Apache2 documentation](https://httpd.apache.org/docs/2.4/). I also used the [Filesystem Hierarchy Standard](http://www.pathname.com/fhs/pub/fhs-2.3.pdf) to further my understanding of the Linux filesystem.

# Initial Setup

Follow Udacity's provided instructions to create your new development instance, courtesy of Amazon's Elastic Cloud. Once you have your private key and public IP address, type into your terminal

    ssh -i ~/.ssh/udacity_key.rsa root@YOURPUBLICIP

where YOURPUBLICIP is the provided public IP address.

# Create new user

You will now create a new user. From the terminal, enter

    adduser grader

We will now make grader a sudo-user by adding him to the sudo group. This is accomplished by entering the following command

    sudo usermod -aG sudo grader

in -aG, the "a" stands for append, and the "G" stands for group. We are using usermod to append the group sudo to grader. There are other ways to make a sudo-user, but this is the easiest and least risky.

## Give user SSH access

On your local machine, navigate to the .ssh directory from the initial setup. We will be generating a public key for our new user. Enter

    ssh-keygen -y

You will be asked for the file in which the private key is, enter

    udacity_key.rsa

The output is the public key we will need to copy. Head back to the virtual machine and enter the following to create save the .ssh public key for user grader. If you logged out of the virtual machine, SSH back in as root and `su grader` to switch to grader.

    mkdir .ssh
    sudo nano .ssh/authorized_keys

Copy the output of `ssh-keygen` into this file, save, and exit. Change the access permissions of these files to complete ssh setup,

    chmod 700 .ssh
    chmod 644 .ssh/authorized_keys

## Confirm access

Attempt to ssh into the server using the new user instead of root to confirm ssh access.

To confirm the user is a sudo-user, enter

    sudo ls -a /root

If you have successfully made a sudo-user you will see a list of directories, otherwise you will get a permission denied error.

# Disable remote root access

Open the sshd_config file in nano (or vim, if you prefer).

    sudo nano /etc/ssh/sshd_config

Change the line

    #PermitRootLogin yes

to

    PermitRootLogin no

1.  At this point as a precaution it can be useful to ssh into the server in another shell, as root, to ensure you are don't lock yourself out of the server. As long as you successfully completed step three, you shouldn't need to worry, but it's better to be safe than sorry. Restart sshd in the terminal.


    sudo /etc/init.d/sshd restart

Grader will now be our admin user, and if we need to switch to root for any reason we can do so by entering

    sudo su root

# Change firewall settings

## SSH configuration

## Update applications

## SSH on non-default

## Configure postgresql

## Ensure remote login works

## Configure to serve item catalog
