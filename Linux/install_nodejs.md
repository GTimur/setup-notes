### How To Install Latest Nodejs on CentOS/RHEL 7/6

#### Add Node.js Yum Repository

##### Latest version:
```
# yum install -y gcc-c++ make
# curl -sL https://rpm.nodesource.com/setup_11.x | sudo -E bash -
```

##### Stable version:
```
yum install -y gcc-c++ make
curl -sL https://rpm.nodesource.com/setup_10.x | sudo -E bash -
````

#### Install Node.js on CentOS
```
sudo yum install nodejs
npm update npm -g

node --version
npm -v
```


### Using NVM (preffered for local install):
```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash
nvm --version

nvm install node
nvm install --lts
nvm ls
nvm use 10.13.0
nvm alias default 10.13.0
```

