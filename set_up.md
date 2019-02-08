# How to set up VPS

I think the best Linux distribution is *Arch Linux*. However, it doesn't be supported by most cloud service providers, so I choose Ubuntu-Server as my OS on the VPS

## Be Careful about Security

### Add normal user

Operate as root could be dangrous, we need to add a normal user in case.

```bash
# add user: make sure you are root
adduser lecoan
usermod -aG sudo lecoan
```

### Disable password login

Use password to login is not a good idea(Espcially when using a week password)

```bash
#set up ssh
su lecoan
cd ~
mkdir .ssh
echo $MY_PUB_KEY >> .ssh/authorized_keys
exit

# close root and password login
vim /etc/ssh/sshd_config
## ...
# turn off root login and password login
## ...
```

### Use Ubuntu Firewall

ufw(Ubuntu firewall)is a great tools for preventing attack by others

```bash
# this operation will allow OpenSSH traffic arrive to the VPS
# If you can learn more my `man ufw` if have other demands
ufw allow OpenSSH
ufw enable
```

## Build Devlop Environment

after those set up above, we can login as a normal user by ssh

### Install Install necessary pacakage

```bash
sudo apt update && sudo apt upgrade -y \
  && sudo apt install zsh vim git tree mlocate python3-pip -y \
  && sudo apt install build-essential cmake python3-dev -y
```

### Use oh-my-zsh as shell

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
vim .zshrc
## ...
# change theme to ys, add z to plugins
## ...
```

### Set up vim for edit

install [amix](https://github.com/amix)/vimrc

```bash
git clone --depth=1 https://github.com/amix/vimrc.git ~/.vim_runtime
sh ~/.vim_runtime/install_awesome_vimrc.sh
```

set up you-complete-me

```bash
cd ~/.vim_runtime/my_plugins
git clone https://github.com/Valloric/YouCompleteMe.git --depth 1
cd YouCompleteMe
sudo pip3 install future
# all options: --ts-completer --java-completer --rust-completer 
#   --go-completer --clang-completer --cs-completer
python3 install.py --clang-completer
```

move my vimrc to configuration

```bash
# TODO
```

## Other Demands

### Set up Shadowsocks Server

if the VPS is located in foreign, a Shadowsocks Server could be helpful

```bash
apt install shadowsocks
vim /etc/shadowsocks/config.json
## ...
# set up your server here
## ...

# we use systemctl to keep the server runs backend
vim /etc/systemd/system/ssserver.service

systemctl start ssserver.service
systemctl enable ssserver.service
```

A example ssserver.service file

```ini
[Unit]
Description=Shadowsocks Server
After=network.target

[Service]
ExecStart=/usr/bin/ssserver -c /etc/shadowsocks/config.json
Restart=on-abort

[Install]
WantedBy=multi-user.target
```

### Generate locale file

```bash
vim /etc/locale.gen
## ...
# uncomment en_US.UTF-8 zh_CN.UTF-8 zh_TW.UTF-8
### ...
locale-gen
```

### Start My Blog Server

I choose hexo to generate HTML files in my computer, and the VPS only need to show those static files using Nginx.

```bash
apt install nginx
# necessary if enable ufw
ufw allow "Nginx HTTP"
ufw allow "Nginx HTTPS"

# create a nginx config file
vim /etc/nginx/sites-available/blog
# enable the config
cd /etc/nginx/sites-enabled
ln -s /etc/nginx/sites-available/blog blog

# space to store static files
mkdir -r /var/www/blog

su lecoan
mkdir ~/hexo.git && cd hexo.git
git init --bare
# edit post-receive hook
vim hooks/post-receive
chmod +x hooks/post-receive
```

A example Nginx config file:

```nginx
server {
	listen 80 default_server;
	listen [::]:80 default_server;

	root /var/www/blog; # where you put your static files
	index index.html index.htm;
	server_name your-host.com;

	location / {
	}
}
```

A example post-receive file:

```bash
GIT_REPO=$HOME/hexo.git
PUBLIC_WWW=/var/www/blog

git --work-tree=${PUBLIC_WWW} --git-dir=${GIT_REPO} checkout -f
```

