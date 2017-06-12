# How To Install Using package manager:
Option 1: 
```
sudo apt-get update
sudo apt-get install build-essential -y
sudo apt-get install nodejs -y
sudo apt-get install npm  -y
```

# How To Install Using NVM

Option 1: 

Install Node.js with Node Version Manager
First, make sure you have a C++ compiler. Open the terminal and install the build-essential and libssl-dev packages if needed. By default, Ubuntu does not come with these tools — but they can be installed in the command line.
```
sudo apt-get install build-essential libssl-dev  -y
```

You can install and update Node Version Manager(Ref: https://github.com/creationix/nvm ), or nvm, by using cURL:

```
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.31.0/install.sh | bash
```
You will be asked to close and reopen the terminal. To verify that nvm has been successfully installed after you reopen the terminal, use:
```
source ~/.profile

nvm ls-remote
nvm install 0.11.13
nvm use 0.11.13
node -v
v.0.11.13
```
if you wish to default one of the versions, you can type:
```
nvm alias default 0.11.13
```
This version will be automatically selected when a new session spawns. You can also reference it by the alias like this:
```
nvm use default
```
