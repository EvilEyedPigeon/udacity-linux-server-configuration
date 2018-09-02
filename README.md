# Udacity Linux Server Configuration

Configuration of a secure Ubuntu 16.04 server on [AWS Lightsail](https://lightsail.aws.amazon.com/) to serve the [League of Legends item catalog Flask application](https://github.com/nginyc/udacity-lol-item-catalog-website).

## Server Configuration Details

Base image is a Ubuntu 16.04 LTS as an AWS Lightsail instance.

- Updated all currently installed packages
- Changed the SSH port from 22 to 2200 for security
- Configured the firewall to deny all incoming connections except for 2200, 80 and 123 for security
- Added a new user `grader` with `sudo` access and key-based authentication
- Disabled remote SSH login with `root` or with cleartext passwords
- Configured local timezone to UTC
- Installed and configured Apache for serving a Python 3 app
- Installed and configured PostgreSQL
- Downloaded, installed and configured the League of Legends item catalog Flask app, including its python dependencies

## Setup Instructions

Prerequisites:

- AWS account
- [Git Bash](https://git-scm.com/downloads), only for Windows machines

1. Create an AWS Lightsail instance with the "Ubuntu 16.04 LTS" blueprint, ensuring that you have the private key of the selected SSH key pair on your machine.

2. SSH into your new Ubuntu server with Terminal (on Mac) or Git Bash (on Windows):

    ```shell
    ssh ubuntu@<instance_public_ip_address> -i <private_key_file>
    ```

    The instance's public IP address can be found under the "Connect" section of your new Lightsail instance's page.

3. Update all currently installed packages.

    Run:

    ```shell
    sudo apt-get update
    sudo apt-get upgrade
    ```

    Next, to set the instance's locale, create file `/etc/default/locale`:
    ```txt
    LANGUAGE=en_US.UTF-8
    LC_ALL=en_US.UTF-8
    LANG=en_US.UTF-8
    LC_TYPE=en_US.UTF-8
    ```

    Reboot the machine:

    ```shell
    sudo reboot
    ```

4. Change the SSH port from 22 to 2200 for security.

    Open the SSH configuration file:

    ```shell
    sudo vim /etc/ssh/sshd_config
    ```

    Update the line `Port 22` to `Port 2200` & save the file.

    Restart the SSH service:

    ```shell
    sudo service sshd restart
    ```

5. Configure the instance's firewall for security.

    Configure and enable the *Uncomplicated Firewall (UFW)*:

    ```shell
    sudo ufw default deny incoming
    sudo ufw default allow outgoing
    sudo ufw allow 2200
    sudo ufw allow 80
    sudo ufw allow 123
    sudo ufw enable
    ```

    Your existing SSH connection would be disrupted as port 22 no longer accepts incoming connections.

    Update your instance's Firewall settings on Lightsail (under "Networking") to allow incoming connections to ports 2200, 80 & 123.

    Then, you can SSH into the server again, this time on port 2200:

    ```shell
    ssh ubuntu@<instance_public_ip_address> -p 2200 -i <private_key_file>
    ```

    Ensure that your firewall is set up properly:

    ```shell
    $ sudo ufw status
    Status: active
    To                         Action      From
    --                         ------      ----
    2200                       ALLOW       Anywhere                  
    80                         ALLOW       Anywhere                  
    123                        ALLOW       Anywhere                  
    2200 (v6)                  ALLOW       Anywhere (v6)             
    80 (v6)                    ALLOW       Anywhere (v6)             
    123 (v6)                   ALLOW       Anywhere (v6)    
    ```

6. Add a new user `grader` with `sudo` access.

    Add the user:

    ```shell
    sudo adduser grader
    ```

    Fill in `grader`'s password & user information.

    Then, create a new file `/etc/sudoers.d/grader`:
    ```txt
    # User rules for grader
    grader ALL=(ALL) NOPASSWD:ALL
    ```

7. Grant `grader` use of key-based authentication:

    Close the SSH session and ensure you are on your local machine's shell.

    ```shell
    exit
    ```

    Generate a new RSA key pair for `grader`, ensuring that you set a passphrase (e.g. `GrAdEr`):

    ```shell
    ssh-keygen -t rsa -b 4096
    ```

    Identify the *public* key file for the generated key pair and copy its contents. The text should look similar to the following:

    ```txt
    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQC8UYch5zYbRQB7LRi7tF29VYDAM8wMf70Zupu34QLTZEdyJrb2w6zu7+bGxb12f5yrEJjah19hmLuIgyYW/PTYkbbeWEEqdo7RP+S4YDWLlVSG+9dtjyY9CqtoBaJgNsHhTTZQacezw8ZvfVq/QywgeERrjgLsXaYJTOwvQfsyTmdO5ebpZPqnuw5V5d5pIGuB6e65i/vdFF5NVFnYmTYR1Gpp+KCR5KFmlX8vp55N1q60SPdXeU44Y48Ze0rcUTxJbUIImOUeHAP57vaBHQhVpNC1aupgllbQoZgEL0xLGLQrducO2u25JYJ5vFKjw0Fp81oIL9KizAI2jSmwt8lAFwP1IiBUe8DO7QqYYJ2H3hNa2kr6UrzA6COR1vdWeDxTeSXlzrPvWl/7JoTWNUhZbmMUuXvwW+K8NBc92uLDdvGIaV2qUxghXya5THfuez8bzLSChvs74NSlxY0m3qv/SCcIS+C5UFysRFUzbU/dNY1HcL5JfSlfvp/fXk6ppkNxXwjSgIpcWww0uFOUYcBB4CBqiYSwliBy6lkcjtf1smNWwIdMNFZqTYic4TWrkFhgaEq1VKInTFyOt/LgOS0SH3dvsJy34cP9pU18wDbh22nzCJ9MDATC+pBWh3x2MvbbxWPzjjY7Uqhj4WxPJxutb2fPLBGqeUf1+6AlWA9g7w== nginyc@Yuns-MacBook-Pro.local
    ```

    Re-establish the SSH connection to your instance and `su` as `grader`:

    ```shell
    su grader
    ```

    Create a file `~/.ssh/authorized_keys` and populate the file with the copied text (as a single line) to allow `grader` to use key-based authentication with the generated key-pair:

    ```shell
    mkdir ~/.ssh
    sudo vim ~/.ssh/authorized_keys
    ```

    The new user `grader` can now do key-based SSH connection to the instance with the *private* key file for the generated key pair:

    ```shell
    ssh grader@<instance_public_ip_address> -p 2200 -i <grader_private_key_file>
    ```

8. Disable remote SSH login with `root` or with cleartext passwords.

    Open the SSH configuration file:

    ```shell
    sudo vim /etc/ssh/sshd_config
    ```

    Edit it such that `PermitRootLogin no` and `PasswordAuthentication no`, then save the file.

    Restart the SSH service:

    ```shell
    sudo service sshd restart
    ```

9. Configure local timezone to UTC.

    Run:

    ```shell
    sudo timedatectl set-timezone UTC
    ```

10. Install Apache & PostgreSQL for serving the Python 3 app as a WSGI app.

    Run:

    ```shell
    sudo apt-get install apache2 libapache2-mod-wsgi
    sudo apt-get install libapache2-mod-wsgi-py3
    sudo apt-get install python3-pip
    sudo apt-get install postgresql
    ```

11. Install the Python app's python dependencies.

    Run:

    ```shell
    sudo pip3 install flask packaging google-api-python-client passlib flask-httpauth
    sudo pip3 install sqlalchemy flask-sqlalchemy psycopg2-binary bleach requests
    ```

12. Configure the Apache2 WSGI app to point to the Python app.

    Edit file `/etc/apache2/sites-enabled/000-default.conf` by adding the following text within the `<VirtualHost></VirtualHost>` tag:

    ```conf
    # Direct requests to Python app & run app as `ubuntu` user
    WSGIDaemonProcess app user=ubuntu group=ubuntu threads=5
    WSGIScriptAlias / /var/www/html/app.wsgi
    ```

    Create a file `/var/www/html/app.wsgi`:

    ```python
    import sys
    import os

    # Read from python modules in project
    sys.path.insert(0, '/home/ubuntu/udacity-lol-item-catalog-website')

    from app import app as application
    ```

    Then restart the apache server:
    ```shell
    sudo apache2ctl restart
    ```

13. Download & configure the item catalog app.

    Clone the project:
    ```shell
    cd ~
    git clone https://github.com/nginyc/udacity-lol-item-catalog-website.git
    ```

    Create a file `~/udacity-lol-item-catalog-website/config.py`:

    ```python
    GOOGLE_OAUTH_CLIENT_ID = '<Google OAuth Client ID>'
    GOOGLE_OAUTH_CLIENT_SECRET = '<Google OAuth Client Secret>'
    APP_SECRET = 'secret'
    DB_URL = 'postgres://catalog:catalog@localhost/catalog'
    ```

14. Intialize the app's PostgreSQL database & seed its data.

    Go into the PSQL console as the `postgres` user:

    ```shell
    sudo -u postgres psql
    ```

    In the PSQL console, create a `catalog` user & `catalog` database, and exit the console:

    ```sql
    CREATE USER catalog WITH PASSWORD 'catalog';
    CREATE DATABASE catalog OWNER catalog;
    \q
    ```

    Run `python3 setup.py` to seed the database with cool League of Legends items:

    ```shell
    cd ~/udacity-lol-item-catalog-website
    python3 setup.py
    ```

15. Visit the site at the instance's public IP address!

## Known Issues

- Google Sign-In throws an error as it restricts OAuth 2.0 from a host with an IP address (rather than a domain). It is likely that getting a domain name for the instance, and subsequently specifying the "origin URI" for your app. Instructions for the integration of Google Sign-In are available [here](https://developers.google.com/identity/sign-in/web/sign-in).

## References

- Configuration of Apache's WSGI: https://modwsgi.readthedocs.io/en/develop/configuration-directives/WSGIDaemonProcess.html
- Appending to `$PATH` within Python: https://stackoverflow.com/questions/16114391/adding-directory-to-sys-path-pythonpath