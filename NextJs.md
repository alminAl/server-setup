# Install Node

```bash
sudo apt update && sudo apt upgrade
```

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

```bash
exec $SHELL
```

```bash
nvm install --lts
```

```bash
apt install npm
```

## Setting up an SSH key

```bash
cd ~/.ssh
```

```bash
ssh-keygen -t ecdsa -b 521 -C "root@<server username>"
```

```bash
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/YOUR_USER/.ssh/id_rsa):
```

If the process is successful, you'll see confirmation messages in the terminal.

- Your identification has been saved in gh_actions
- Your public key has been saved in gh_actions.pub

- Your identification has been saved in gh_actions
- Your public key has been saved in gh_actions.pub

```bash
cat gh_actions.pub >> ~/.ssh/authorized_keys
```

With the newly generated key in place, it's now possible to log into the server.

The final step involves copying the generated private key to the clipboard. This will be essential when establishing the secrets for GitHub Actions.

```bash
cat ~/.ssh/gh_actions
```

## Setting up secrets to enable GitHub Actions deployment

In order to allow GitHub Actions to establish a connection with our server without revealing sensitive information, we'll house this data in GitHub Actions secrets. Follow these steps to set them up:

Navigate to your GitHub repository and select the Settings tab.
Locate the Secrets and actions dropdown and click to expand it.
Click on the Actions tab, which takes you to the Actions secrets and variables page.
Towards the top left, find and click on the New repository secret button to initiate the creation of new secrets.
You'll need to create the following secrets:

- SSH_KEY: The private SSH key we copied earlier.
- HOST: The domain name or IP address of the deployment server.
- USERNAME: The user for whom the SSH key was created.
- TARGET_DIRECTORY: The destination for the deployed code. For DigitalOcean, this typically is /var/www/yoursitename.

After creating these secrets, we'll have gathered all the necessary information to establish the deployment workflow.

## Process manager

```bash
#Go to site directory and launch it with pm2
cd /var/www/name_of_app

#launch app with pm2
pm2 start pnpm --name name_of_app -- start -p 3000
```

## Nginx

```bash
#nginx config file for Nextjs App
#place in /etc/nginx/sites-available/name_of_config_file
server {
        server_name krishijaat.com www.krishijaat.com;

        gzip on;
        gzip_proxied any;
        gzip_types application/javascript application/x-javascript text/css text/javascript;
        gzip_comp_level 5;
        gzip_buffers 16 8k;
        gzip_min_length 256;

        location /_next/static/ {
                alias /home/krishijaat/krishijaat/.next/static/;
                expires 365d;
                access_log off;
        }

        location / {
                proxy_pass http://127.0.0.1:3001; #change to 3001 for second app, but make sure second nextjs app starts on new port in packages.json "start": "next start -p 3001",
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection 'upgrade';
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
        }
}
```
