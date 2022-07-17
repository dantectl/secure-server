# secure-server
This script can assist in automating some best security practices to `secure your server`. This should work with most Debian based systems. The process does require some manual entry, and is somewhat a work in progress, but I hope it helps!

Here's a high-level breakdown of the sections, the full script can be found below which you can copy for your own use.

`Prerequisite`
Upload your public key to the following directory:
```
/root/bin
```
### Update your system
```
apt-get update --yes && apt-get upgrade --yes
```

### Set hostname and timezone
```
hostnamectl set-hostname <your-host-name>
timedatectl set-timezone <your-preferred-timezone>
```

reference - [timedatectl](https://www.freedesktop.org/software/systemd/man/timedatectl.html#) 

### Harden SSH
```
sed -i -e 's/#PasswordAuthentication yes/PasswordAuthentication no/g' /etc/ssh/sshd_config
sed -i -e 's/PermitRootLogin yes/PermitRootLogin no/g' /etc/ssh/sshd_config

# this command changes the default ssh port, you may delete this line if you want to keep port 22
sed -i -e 's/#Port 22/Port <your-preferred-port-number>/g' /etc/ssh/sshd_config
sudo systemctl restart sshd -y
sudo service ssh restart -y
```

### Enable the firewall
```
ufw enable
ufw logging on
ufw allow <specified-ssh-port>
```

Ok, so here's where the manual part comes into play (setting the user password)


### Add limited user
```
adduser <limited-username>
adduser <user> sudo
su - <user> bash -c 'mkdir -p ~/.ssh && chmod -R 700 ~/.ssh/'
cp ~/bin/<your_rsa.pub> /home/<user>/.ssh/authorized_keys
chown <user>:<user> /home/<user>/.ssh/authorized_keys
su - <user> bash -c 'chmod 644 ~/.ssh/authorized_keys'
```

### Install fail2ban
```
apt-get install fail2ban
cp /etc/fail2ban/fail2ban.conf /etc/fail2ban/fail2ban.local
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

### Full script
```
#!/bin/bash
apt-get update --yes && apt-get upgrade --yes
hostnamectl set-hostname <hostname>
timedatectl set-timezone <enter-your-region>
sed -i -e 's/#PasswordAuthentication yes/PasswordAuthentication no/g' /etc/ssh/sshd_config
sed -i -e 's/PermitRootLogin yes/PermitRootLogin no/g' /etc/ssh/sshd_config

# this command changes the default ssh port, you may delete this line if you want to keep port 22
sed -i -e 's/#Port 22/Port <port#>/g' /etc/ssh/sshd_config

sudo systemctl restart sshd -y
sudo service ssh restart -y
ufw enable
ufw logging on
ufw allow <uncommon-port-number>
adduser <user>
adduser <user> sudo
su - <user> bash -c 'mkdir -p ~/.ssh && chmod -R 700 ~/.ssh/'
cp ~/bin/<your_rsa>.pub /home/<user>/.ssh/authorized_keys
chown <user>:<user> /home/<user>/.ssh/authorized_keys
su - <user> bash -c 'chmod 644 ~/.ssh/authorized_keys'
apt-get install fail2ban
cp /etc/fail2ban/fail2ban.conf /etc/fail2ban/fail2ban.local
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```
