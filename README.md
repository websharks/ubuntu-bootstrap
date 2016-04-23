## VirtualBox + Vagrant + Landrush; running Ubuntu w/ Nginx (or Apache), MariaDB (MySQL), PHP (choice of custom 7.0+, 7.0, 5.6, 5.5), and WordPress

![](http://cdn.websharks-inc.com/jaswsinc/uploads/2015/03/os-x-vagrant-virtualbox.png)

---

### Installation Instructions

#### Step 1: Satisfy Software Requirements

You need to have VirtualBox, Vagrant, and Landrush installed.

```bash
$ brew cask install virtualbox
$ brew cask install vagrant
$ vagrant plugin install landrush

$ vagrant plugin install vagrant-cachier # Suggested (optional).
$ vagrant plugin install vagrant-triggers # Suggested (optional).
```

You need to install the `ubuntu/trusty64` Box.

```bash
$ vagrant box add ubuntu/trusty64
```

#### Step 2: Clone GitHub Repo (Ubuntu Bootstrap)

```bash
$ mkdir ~/VMs && cd ~/VMs
$ git clone https://github.com/websharks/ubuntu-bootstrap my.vm
$ cd ~/VMs/my.vm && git lfs pull # Pull large file storage.
```

_Note that `my.vm` becomes your domain name. Change it if you like. Must end with `.vm` please._

#### Step 3: Vagrant Up!

```bash
$ cd ~/VMs/my.vm
$ vagrant up
```

#### Step 4: Install software :-)

```bash
$ vagrant ssh
$ sudo /bootstrap/src/installer # Presents a configuration dialog.
# Tip: to bypass configuration add the `--CFG_USE_WIZARD=0` argument to the installer.
```

#### Step 5: Confirm it is Working!

- Open <http://my.vm>. You should see a WordPress installation page.
- Open <https://my.vm>. You should get an SSL security warning. Please bypass this self-signed certificate warning and proceed. You should again see the WordPress installation page. SSL is working as expected!

  _The URL <http://my.vm> is not working?_

  _Try flushing your DNS cache. Each time you `vagrant up`, a new IP is generated automatically that is mapped to the `my.vm` hostname. If you are working with multiple VMs, you might need to flush your DNS cache to make sure your system is mapping `my.vm` to the correct IP address. See: <http://jas.xyz/1fmAa4P> for instructions on a Mac._

---

### Additional Steps (All Optional)

#### Step 6: Add Files to: `~/VMs/my.vm/app/src/`

The is the web root. The latest version of WordPress will already be installed. However, you can add any additional application files that you'd like. e.g., phpBB, Drupal, Joomla, whatever you like. It's probably a good idea to put anything new inside a sub-directory of its own; e.g., `~/VMs/my.vm/app/src/phpBB`

#### Step 7: Understanding Environment Variables

This stack comes preconfigured with a MySQL database and environment variables you can use in any PHP config. files.

- `$_SERVER['CFG_MYSQL_DB_HOST']` This is the database host name. Defaults to `127.0.0.1`. Port is `3306` (default port).
- `$_SERVER['CFG_MYSQL_DB_NAME']` This is the database name. Defaults to `admin`.
- `$_SERVER['CFG_MYSQL_DB_USERNAME']` This is the database username. Defaults to `admin`.
- `$_SERVER['CFG_MYSQL_DB_PASSWORD']` This is the database password. Defaults to `admin`.

_**Tip:** For a full list of all global environment variables, see: `setups/env-vars` in the repo._

#### Step 8: Learn to Use the Tools That I've Bundled

A username/password is required to access each of these tools. It is always the same thing.

- Username: `admin` Password: `admin`

Available Tools (Using Any of These is Optional):

- <https://my.vm/---tools/pma> PhpMyAdmin
  DB name: `admin`, DB username: `admin`, DB password: `admin`
- <https://my.vm/---tools/opcache.php> PHP OPcache extension status dump.
- <https://my.vm/---tools/info.php> `phpinfo()` output page.
- <https://my.vm/---tools/fpm-status.php> PHP-FPM status page.
- <https://my.vm/---tools/status.nginx> NGINX status page (if Nginx was installed).
- <https://my.vm/---tools/apache-status> Apache status page (if Apache was installed).
- <https://my.vm/---tools/apache-info> Apache page (if Apache was installed).

#### Step 9: Tear it Down and Customize

```bash
$ cd ~/VMs/my.vm
$ vagrant destroy
```

In the project directory you'll find a `/src/vagrant/bootstrap-custom` file. This bash script runs as the `root` user during `vagrant up`. Therefore, you can install software and configure anything you like in this script. By default, this script does nothing. All of the software installation and system configuration takes place whenever you run `/bootstrap/src/installer` inside the VM.

##### Customization (Two Choices Available)

1. Customize `/src/installer` and the associated setup files that it calls upon, which are located in: `/src/setups/*`. _**Note:** If you go this route, there really is no reason to customize the `/src/vagrant/bootstrap-custom` file. You can leave it as-is._

2. Or, instead of working with the more complex installer, you can keep things simple and add your customizations to the `/src/vagrant/bootstrap-custom` script, which is a very simple starting point. The `/src/vagrant/bootstrap-custom` runs whenever you type `vagrant up`, so this is a logical choice for beginners. _**Note:** If you go this route, you can simply choose not to run `/bootstrap/src/installer`, because all of your customizations will be in the `/src/vagrant/bootstrap-custom` file; i.e., there will be no reason to run the installer._

##### When you're done with your customizations, type:

```bash
$ vagrant up
```

###### If you decided to use the `/bootstrap/src/installer` option, also type:

```bash
$ vagrant ssh
$ sudo /bootstrap/src/installer # Presents a configuration dialog.
# Tip: to bypass configuration add the `--CFG_USE_WIZARD=0` argument to the installer.
```

---

### Domain Name Tips & Tricks

#### Creating a Second VM w/ a Different Domain Name

```bash
$ git clone https://github.com/jaswsinc/vagrant-ubuntu-lemp my-second.vm
$ cd my-second.vm
$ vagrant up
$ vagrant ssh
$ sudo /bootstrap/src/installer # Presents a configuration dialog.
# Tip: to bypass configuration add the `--CFG_USE_WIZARD=0` argument to the installer.
```

#### Understanding Domain Name Mapping

The URL which leads to your VM is based on the name of the directory that you cloned the repo into; e.g., `my.vm` or `my-second.vm` in the above examples. However, the directory that you clone into MUST end with `.vm` for this to work as expected. If the directory you cloned into doesn't end with `.vm`, the default domain name will be `http://ubuntu.vm`. You can change this hard-coded default by editing `config.vm.hostname` in `Vagrantfile`.

In either case, the domain name is also wildcarded; i.e., `my.vm`, `www.my.vm`, `wordpress.my.vm` all map to the exact same location: `~/VMs/my.vm/app/src/`. This is helpful when testing WordPress Multisite Networks, because you can easily setup a sub-domain network, or even an MU domain mapping plugin.

---

### Testing WordPress Themes/Plugins Easily!

See `/Vagrantfile` where you will find this section already implemented.
_~ See also: `/src/setups/wordpress`_

```ruby
# Mount WordPress projects directory.
if File.directory?(wp_projects_dir = ENV['WP_PROJECTS_DIR'] || File.expand_path('~/projects/wordpress'))
  config.vm.synced_folder wp_projects_dir, '/wordpress', mount_options: ['ro'];
end;

# Mount WordPress personal projects directory.
if File.directory?(wp_personal_projects_dir = ENV['WP_PERSONAL_PROJECTS_DIR'] || File.expand_path('~/projects/personal/wordpress'))
  config.vm.synced_folder wp_personal_projects_dir, '/wp-personal', mount_options: ['ro'];
end;

# Mount WordPress business projects directory.
if File.directory?(wp_business_projects_dir = ENV['WP_BUSINESS_PROJECTS_DIR'] || File.expand_path('~/projects/business/wordpress'))
  config.vm.synced_folder wp_business_projects_dir, '/wp-business', mount_options: ['ro'];
end;
```

#### ↑ What is happening here w/ these WordPress directories?

The `Vagrantfile` is automatically mounting drives on your VM that are sourced by your local `~/projects` directory (if you have one). Thus, if you have your WordPress themes/plugins in `~/projects/wordpress` (i.e., in your local filesystem), it will be mounted on the VM automatically, as `/wordpress`.

In the `src/setups/wordpress` file, we iterate `/wordpress` and build symlinks for each of your themes/plugins automatically. This means that when you log into your WordPress Dashboard (<http://my.vm/wp-admin/>), you will have all of your themes/plugins available for testing. If you make edits locally in your favorite editor, they are updated in real-time on the VM. Very cool!

The additional mounts (i.e., `~/projects/personal/wordpress` and `~/projects/business/wordpress`) are simply alternate locations that I use personally. Remove them if you like. See: `Vagrantfile` and `src/setups/wordpress` to remove in both places. You don't really _need_ to remove them though, because if these locations don't exist on your system they simply will not be mounted. In fact, you might consider leaving them, and just alter the paths to reflect your own personal preference—or for future implementation.

#### The default WordPress mapping looks like this:

- `~/projects/wordpress` on your local system.
  - Is mounted on the VM, as: `/wordpress`
- Then (on the VM) the `/src/setups/wordpress` script symlinks each theme/plugin into:
  - `/app/src/wp-content/[themes|plugins]` appropriately.

#### What directory structure do I need exactly?

Inside `~/projects/wordpress` you need to have two sub-directories. One for themes and another for plugins.

- `~/projects/wordpress/themes` (put WP themes in this directory; e.g., `my-theme`)
- `~/projects/wordpress/plugins` (put WP plugins here; e.g., `my-plugin`)

Now, whenever you run `/bootstrap/src/installer` from the VM, your local copy of `~/projects/wordpress/themes/my-theme` becomes `/app/src/wp-content/themes/my-theme` on the VM. Your local copy of `~/projects/wordpress/plugins/my-plugin` becomes `/app/src/wp-content/plugins/my-plugin` on the VM ... and so on... for each theme/plugin sub-directory, and for each of the three possible mounts listed above. This all happens automatically if you followed the instructions correctly.

#### Can I override the default source directories for WordPress?

Yes. Looking over the code snippet above, you can see that there are three environment variables that you can set in your `~/.profile` that (if found) will override the default locations automatically. Here's a quick example showing how you might customize these in your own `~/.profile`.

```bash
export WP_PROJECTS_DIR=~/my-projects/wordpress;
export WP_PERSONAL_PROJECTS_DIR=~/my-personal-projects/wordpress;
export WP_BUSINESS_PROJECTS_DIR=~/my-business-projects/wordpress;
```

---

### Bootstrap Command-Line Arguments

The following CLI arguments can be passed to `/bootstrap/src/installer`, just in case you'd like to avoid the configuration wizard entirely. These are all optional; i.e., if you don't provide these arguments you will be prompted to configure the bootstrap using a command-line dialog interface whenever the installer runs.

_**Tip:** You can learn more about how these work and what the defaults are by looking over the [src/setups/config](src/setups/config) file carefully and perhaps searching for their use in other files found in `src/setups/*`._

- `--CFG_USE_WIZARD=0|1` `0` to bypass the wizard.

---

- `--CFG_4CI=0|1` Building this as a base image for a CI server?
- `--CFG_4PKG=0|1` Building this as a base image that will be packaged up?

---

- `--CFG_HOST=my.cool.vm` Host name. Normally this is just `my.vm`.
- `--CFG_ROOT_HOST=cool.vm` Root host name. Normally this is the same as `CFG_HOST`.
- `--CFG_OTHER_HOSTS=[comma-delimited]` e.g., `a.vm,b.vm,c.vm` Other hosts to resolve locally. These are only resolved inside the VM; i.e., these additional host names are added to `/etc/hosts` inside the VM so the VM will be capable of connecting to them. Mainly useful when building a base image for a CI server that is entirely self-contained.

---

- `--CFG_SLUG=my-cool-vm` Slug w/ dashes, no underscores.
- `--CFG_VAR=my_cool_vm` Slug w/ underscores, no dashes.

---

- `--CFG_ADMIN_USERNAME=admin` Administrative username.
- `--CFG_ADMIN_PASSWORD=admin` Administrative password.
- `--CFG_ADMIN_NAME='Admin Istrator'` Display name.
- `--CFG_ADMIN_EMAIL='admin@my.cool.vm'` Admin email address.
- `--CFG_ADMIN_PUBLIC_EMAIL='hostnamster@my.cool.vm'` Public email address.
- `--CFG_ADMIN_PREFERRED_SHELL=/bin/zsh` Or `/bin/bash`.
- `--CFG_ADMIN_AUTHORIZED_SSH_KEYS=` e.g., `/authorized_keys`

---

- `--CFG_TOOLS_USERNAME=admin` Administrative username.
- `--CFG_TOOLS_PASSWORD=admin` Administrative password.
- `--CFG_TOOLS_PMA_BLOWFISH_KEY=[key]` Secret key. Default is auto-generated.
- `--CFG_MAINTENANCE_BYPASS_KEY=[key]` Secret key. Default is auto-generated.

---

- `--CFG_MYSQL_DB_HOST=127.0.0.1` MySQL DB host name.
- `--CFG_MYSQL_DB_PORT=3306` MySQL DB port number.
- `--CFG_MYSQL_SSL_ENABLE=0|1` MySQL DB supports SSL connections?
- `--CFG_MYSQL_DB_CHARSET=utf8mb4` MySQL DB charset.
- `--CFG_MYSQL_DB_COLLATE=utf8mb4_unicode_ci` MySQL DB collation.
- `--CFG_MYSQL_DB_NAME=db0` MySQL DB name.
- `--CFG_MYSQL_DB_USERNAME=client` MySQL DB username.
- `--CFG_MYSQL_DB_PASSWORD=[key]` Default is auto-generated.

---

- `--CFG_INSTALL_SWAP=0|1` Install swap space?

---

- `--CFG_INSTALL_WS_CA_FILES=0|1` Use WebSharks certificate authority?

---

- `--CFG_INSTALL_DOCKER=0|1` Install Docker?

---

- `--CFG_INSTALL_POSTFIX=0|1` Install Postfix?

---

- `--CFG_INSTALL_MAILCATCHER=0|1` Install Mailcatcher?

---

- `--CFG_INSTALL_NGINX=0|1` Install Nginx?
- `--CFG_INSTALL_APACHE=0|1` Install Apache?

---

- `--CFG_INSTALL_MYSQL=0|1` Install MySQL?
- `--CFG_INSTALL_MYSQL_DB_IMPORT_FILE=[file path on the VM]` Import SQL tables?

---

- `--CFG_INSTALL_MEMCACHE=0|1` Install Memcached?
- `--CFG_INSTALL_RAMDISK=0|1` Install a RAM disk partition?

---

- `--CFG_INSTALL_PHP_CLI=0|1` Install PHP command-line interpreter?
- `--CFG_INSTALL_PHP_FPM=0|1` Install PHP-FPM process manager for Apache/Nginx?
- `--CFG_INSTALL_PHP_VERSION=[custom-src|custom|7.0|5.6|5.5]` Which PHP version?

---

- `--CFG_INSTALL_PHPCS=0|1` Install PHP Code Sniffer?
- `--CFG_INSTALL_PHING=0|1` Install Phing?
- `--CFG_INSTALL_PHPUNIT=0|1` Install PHPUnit?
- `--CFG_INSTALL_APIGEN=0|1` Install APIGen?
- `--CFG_INSTALL_CASPERJS=0|1` Install CasperJS?
- `--CFG_INSTALL_COMPOSER=0|1` Install Composer?
- `--CFG_INSTALL_WP_CLI=0|1` Install WP CLI tool?
- `--CFG_INSTALL_WP_I18N_TOOLS=0|1` Install WP i18n tools?

---

- `--CFG_INSTALL_APP_REPO=0|1` Install a Git repo for the `/app/src` directory?

---

- `--CFG_INSTALL_DISCOURSE=0|1` Install Discourse?
- `--CFG_DISCOURSE_SMTP_HOST=email-smtp.us-east-1.amazonaws.com` SMTP host name.
- `--CFG_DISCOURSE_SMTP_PORT=587` SMTP port number.
- `--CFG_DISCOURSE_SMTP_AUTH_TYPE=login` SMTP authentication type.
- `--CFG_DISCOURSE_SMTP_USERNAME=[username]` SMTP username.
- `--CFG_DISCOURSE_SMTP_PASSWORD=[password]` SMTP password.

---

- `--CFG_INSTALL_WORDPRESS=0|1` Install WordPress?
- `--CFG_INSTALL_WORDPRESS_VM_SYMLINKS=0|1` Install WordPress theme/plugin symlinks?

---

- `--CFG_INSTALL_FIREWALL=0|1` Install firewall?
- `--CFG_INSTALL_FAIL2BAN=0|1` Install Fail2Ban?
- `--CFG_FIREWALL_ALLOWS_MYSQL_INSIDE_VPN=0|1` Allow network-based connections?
- `--CFG_FIREWALL_ALLOWS_CF_ONLY_VIA_80_443=0|1` Allow only CloudFlare?

---

- `--CFG_INSTALL_UNATTENDED_UPGRADES=0|1` Configure unattended upgrades?

---

- `--CFG_CONFIG_FILE=/app/.config.json` This is unrelated to the installer. It's for any purpose you like. e.g., To help you configure a web-based application that will run on this VM. The value that you set here becomes a global environment variable that your application can consume in any way you like. It is expected to be a config file path on the VM.

---

#### Bootstrapping Base Images for Custom Vagrant Boxes

When building a base image that is going to be repackaged, please be sure to disable the Vagrant Landrush plugin beforehand; i.e., you don't want its configuration to become a part of any base image. Disabling Landrush can be accomplished by exporting the following environment variable before you run `$ vagrant up`. See [`Vagrantfile`](Vagrantfile) for details.

```bash
$ export VM_4PKG=1;
$ vagrant up
```

After running `$ vagrant up`, SSH into the VM and run `$ /bootstrap/src/installer`. The installer itself will also be aware of the `VM_4PKG` environment variable, which enables repackaging-specific routines in the bootstrap installer; i.e.., better defaults, additional cleanups, drive compression, etc.

```bash
$ sudo /bootstrap/src/installer --CFG_USE_WIZARD=0 --CFG_INSTALL_PHP_VERSION=7.0;
```
