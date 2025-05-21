## Generate a new ssh key in your machine
```
ssh-keygen -t ed25519 -f ~/.ssh/<KEY_NAME> -C "some comment (optional)"
```

## Access the remote server and create a non root user
```
ssh root@<SERVER_IP>
```
- Create a `.sh` file inside `/home` directory with the contents to create a new 'non root' user. The content can be found [here](https://gist.github.com/fczanetti/64956547ab4468bcfbb3c714801bb55e);

- Execute the file to create the non root user;
```
/home/<FILE>.sh
```

- Log out and log in with the non root user. If necessary, use the `-i` option to specify which ssh file should be used to authenticate;
```
ssh -i ~/.ssh/NEW_SSH_FILE <NEW_USERNAME>@<SERVER_IP>
```

## Clone the repository from GitHub
- Clone the repository from GitHub to the new server. It will be necessary to create a new ssh key inside the server and add it to GitHub. If the 'permission denied' error arises, use the following commands to create a config file and try again;
- Check if the connection with GitHub via ssh is OK:
```
ssh -T git@github.com
```
- Check if the config file already exists. If not:
```
touch ~/.ssh/config
```
- Place it inside the new config file:
```
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/<NEW_SSH_FILE>
```

## Disable login via root user
- To disable login in our server via root user, we can edit the `PermitRootLogin` setting in the `sshd_config` file. This file is usually located in `/etc/ssh`:
```
vim /etc/ssh/sshd_config
```
- find the `PermitRootLogin no` setting and change it to `PermitRootLogin no`;
- for the changes to be applied, restart the ssh service:
```
sudo systemctl restart ssh
```

## Install Docker
We can follow [this tutorial](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-20-04) from Digital Ocean to install Docker in our remote server;

## Set a new firewall
- get the public IP of your computer:
```
curl ifconfig.me
```

- Set the following inbound rules on the firewall. If others are going to access the server, their IP's have to be added in the sources. After setting it up, only the listed IP's will be able to connect to the server.

| Type      | Protocol | Port       | Sources |
|-----------|-------|--------------|--------|
| SSH      | TCP    | 22 | <YOUR_MACHINE_PUBLIC_IP>  # ssh connection |
| HTTP     | TCP    | 80 | <YOUR_MACHINE_PUBLIC_IP>  # standard http |
| HTTPS     | TCP    | 443 | <YOUR_MACHINE_PUBLIC_IP> |
| Custom     | TCP    | 5432 | <YOUR_MACHINE_PUBLIC_IP>  # postgresql |
| Custom     | TCP    | 5678 | <YOUR_MACHINE_PUBLIC_IP>  # n8n |

The 5678 port can be removed after setting up Nginx as a proxy.

## Installing a proxy
We can use Nginx to work as a proxy for us. The docker-compose file was updated to have a new container running on the same network as the n8n container. We also created a new file called `n8n/nginx.conf`, and this is the file where we set the proxy to forward the requests to n8n container. As the Nginx is the one who is going to listen to the requests on ports 80 and 443, we don't have to expose the ports `5678:5678` in n8n container anymore. 