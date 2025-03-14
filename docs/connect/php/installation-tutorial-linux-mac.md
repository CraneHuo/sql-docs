---
title: "Linux and macOS Installation for the Drivers for PHP"
description: "In these instructions, learn how to install the Microsoft Drivers for PHP for SQL Server on Linux or macOS."
author: David-Engel
ms.author: v-davidengel
ms.date: 03/06/2023
ms.service: sql
ms.subservice: connectivity
ms.topic: conceptual
ms.custom: intro-installation
---

# Linux and macOS Installation Tutorial for the Microsoft Drivers for PHP for SQL Server

The following instructions assume a clean environment and show how to install PHP 8.1, the Microsoft ODBC driver, the Apache web server, and the Microsoft Drivers for PHP for SQL Server on Ubuntu, Red Hat, Debian, SUSE, Alpine, and macOS. These instructions advise installing the drivers using PECL, but you can also download the prebuilt binaries from the [Microsoft Drivers for PHP for SQL Server](https://github.com/Microsoft/msphpsql/releases) GitHub project page and install them following the instructions in [Loading the Microsoft Drivers for PHP for SQL Server](../../connect/php/loading-the-php-sql-driver.md). For an explanation of extension loading and why we do not add the extensions to php.ini, see the section on [loading the drivers](../../connect/php/loading-the-php-sql-driver.md#loading-the-driver-at-php-startup).

The following instructions install PHP 8.1 by default using `pecl install`, if the PHP 8.1 packages are available. You may need to run `pecl channel-update pecl.php.net` first. Some supported Linux distros default to PHP 7.1 or earlier, which is not supported for the latest version of the PHP drivers for SQL Server. See the notes at the beginning of each section to install PHP 8.0 or 8.2 instead.

Also included are instructions for installing the PHP FastCGI Process Manager, PHP-FPM, on Ubuntu. PHP-FPM is needed if you're using the nginx web server instead of Apache.

While these instructions contain commands to install both SQLSRV and PDO_SQLSRV drivers, the drivers can be installed and function independently. Users comfortable with customizing their configuration can adjust these instructions to be specific to SQLSRV or PDO_SQLSRV. Both drivers have the same dependencies except where noted below.

Please refer to [Support Matrix](microsoft-php-drivers-for-sql-server-support-matrix.md) for the latest supported operating systems version.

> [!NOTE]
> Make sure you have installed the latest ODBC driver version to ensure optimal performance and security. For installation instructions, see [Install the Microsoft ODBC driver for SQL Server (Linux)](../odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server.md) or [Install the Microsoft ODBC driver for SQL Server (macOS)](../odbc/linux-mac/install-microsoft-odbc-driver-sql-server-macos.md).

## Installing on Ubuntu


> [!NOTE]
> To install PHP 8.0 or 8.2, replace 8.1 with 8.0 or 8.2 in the following commands.

### Step 1. Install PHP (Ubuntu)

```bash
sudo su
add-apt-repository ppa:ondrej/php -y
apt-get update
apt-get install php8.1 php8.1-dev php8.1-xml -y --allow-unauthenticated
```

### Step 2. Install prerequisites (Ubuntu)

Install the ODBC driver for Ubuntu by following the instructions on the [Install the Microsoft ODBC driver for SQL Server (Linux)](../../connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server.md). Make sure to also install the `unixodbc-dev` package. It's used by the `pecl` command to install the PHP drivers.

```bash
sudo apt-get install unixodbc-dev
```

### Step 3. Install the PHP drivers for Microsoft SQL Server (Ubuntu)

```bash
sudo pecl install sqlsrv
sudo pecl install pdo_sqlsrv
sudo su
printf "; priority=20\nextension=sqlsrv.so\n" > /etc/php/8.1/mods-available/sqlsrv.ini
printf "; priority=30\nextension=pdo_sqlsrv.so\n" > /etc/php/8.1/mods-available/pdo_sqlsrv.ini
exit
sudo phpenmod -v 8.1 sqlsrv pdo_sqlsrv
```

If there is only one PHP version in the system, then the last step can be simplified to `phpenmod sqlsrv pdo_sqlsrv`.

### Step 4. Install Apache and configure driver loading (Ubuntu)

```bash
sudo su
apt-get install libapache2-mod-php8.1 apache2
a2dismod mpm_event
a2enmod mpm_prefork
a2enmod php8.1
exit
```

### Step 5. Restart Apache and test the sample script (Ubuntu)

```bash
sudo service apache2 restart
```

To test your installation, see [Testing your installation](#testing-your-installation) at the end of this document.

## Installing on Ubuntu with PHP-FPM


> [!NOTE]
> To install PHP 8.0 or 8.2, replace 8.1 with 8.0 or 8.2 in the following commands.

### Step 1. Install PHP (Ubuntu with PHP-FPM)

```bash
sudo su
add-apt-repository ppa:ondrej/php -y
apt-get update
apt-get install php8.1 php8.1-dev php8.1-fpm php8.1-xml -y --allow-unauthenticated
```

Verify the status of the PHP-FPM service by running:

```bash
systemctl status php8.1-fpm
```

### Step 2. Install prerequisites (Ubuntu with PHP-FPM)

Install the ODBC driver for Ubuntu by following the instructions on the [Install the Microsoft ODBC driver for SQL Server (Linux)](../../connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server.md). Make sure to also install the `unixodbc-dev` package. It's used by the `pecl` command to install the PHP drivers.

### Step 3. Install the PHP drivers for Microsoft SQL Server (Ubuntu with PHP-FPM)

```bash
sudo pecl config-set php_ini /etc/php/8.1/fpm/php.ini
sudo pecl install sqlsrv
sudo pecl install pdo_sqlsrv
sudo su
printf "; priority=20\nextension=sqlsrv.so\n" > /etc/php/8.1/mods-available/sqlsrv.ini
printf "; priority=30\nextension=pdo_sqlsrv.so\n" > /etc/php/8.1/mods-available/pdo_sqlsrv.ini
exit
sudo phpenmod -v 8.1 sqlsrv pdo_sqlsrv
```

If there is only one PHP version in the system, then the last step can be simplified to `phpenmod sqlsrv pdo_sqlsrv`.

Verify that `sqlsrv.ini` and `pdo_sqlsrv.ini` are located in `/etc/php/8.1/fpm/conf.d/`:

```bash
ls /etc/php/8.1/fpm/conf.d/*sqlsrv.ini
```

Restart the PHP-FPM service:

```bash
sudo systemctl restart php8.1-fpm
```

### Step 4. Install and configure nginx (Ubuntu with PHP-FPM)

```bash
sudo apt-get update
sudo apt-get install nginx
sudo systemctl status nginx
```

To configure nginx, you must edit the `/etc/nginx/sites-available/default` file. Add `index.php` to the list below the section that says `# Add index.php to the list if you are using PHP`:

```config
# Add index.php to the list if you are using PHP
index index.html index.htm index.nginx-debian.html index.php;
```

Next, uncomment and modify the section following `# pass PHP scripts to FastCGI server` as follows:

```config
# pass PHP scripts to FastCGI server
#
location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
}
```

### Step 5. Restart nginx and test the sample script (Ubuntu with PHP-FPM)

```bash
sudo systemctl restart nginx.service
```

To test your installation, see [Testing your installation](#testing-your-installation) at the end of this document.

## Installing on Red Hat


### Step 1. Install PHP (Red Hat)

To install PHP on Red Hat 7, run the following commands:
> [!NOTE]
> To install PHP 8.0 or 8.2, replace remi-php81 with remi-php80 or remi-php82 respectively in the following commands.

```bash
sudo su
yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
yum install https://rpms.remirepo.net/enterprise/remi-release-7.rpm
subscription-manager repos --enable=rhel-7-server-optional-rpms
yum install yum-utils
yum-config-manager --enable remi-php81
yum update
# Note: The php-pdo package is required only for the PDO_SQLSRV driver
yum install php php-pdo php-pear php-devel
```

To install PHP on Red Hat 8, run the following commands:
> [!NOTE]
> To install PHP 8.0 or 8.2, replace remi-8.1 with remi-8.0 or remi-8.2 respectively in the following commands.

```bash
sudo su
dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
dnf install yum-utils
dnf module reset php
dnf module install php:remi-8.1
subscription-manager repos --enable codeready-builder-for-rhel-8-x86_64-rpms
dnf update
# Note: The php-pdo package is required only for the PDO_SQLSRV driver
dnf install php-pdo php-pear php-devel
```

### Step 2. Install prerequisites (Red Hat)

Install the ODBC driver for Red Hat 7 or 8 by following the instructions on the [Install the Microsoft ODBC driver for SQL Server (Linux)](../../connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server.md). Make sure to also install the `unixodbc-dev` package. It's used by the `pecl` command to install the PHP drivers.

### Step 3. Install the PHP drivers for Microsoft SQL Server (Red Hat)

```bash
sudo pecl install sqlsrv
sudo pecl install pdo_sqlsrv
sudo su
echo extension=pdo_sqlsrv.so >> `php --ini | grep "Scan for additional .ini files" | sed -e "s|.*:\s*||"`/30-pdo_sqlsrv.ini
echo extension=sqlsrv.so >> `php --ini | grep "Scan for additional .ini files" | sed -e "s|.*:\s*||"`/20-sqlsrv.ini
exit
```

You can alternatively install from the Remi repo:

```bash
sudo yum install php-sqlsrv
```

### Step 4. Install Apache (Red Hat)

```bash
sudo yum install httpd
```

SELinux is installed by default and runs in Enforcing mode. To allow Apache to connect to databases through SELinux, run the following command:

```bash
sudo setsebool -P httpd_can_network_connect_db 1
```

### Step 5. Restart Apache and test the sample script (Red Hat)

```bash
sudo apachectl restart
```

To test your installation, see [Testing your installation](#testing-your-installation) at the end of this document.

## Installing on Debian


> [!NOTE]
> To install PHP 8.0 or 8.2, replace 8.1 in the following commands with 8.0 or 8.2.

### Step 1. Install PHP (Debian)

```bash
sudo su
apt-get install curl apt-transport-https
wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg
echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/php.list
apt-get update
apt-get install -y php8.1 php8.1-dev php8.1-xml php8.1-intl
```

### Step 2. Install prerequisites (Debian)

Install the ODBC driver for Debian by following the instructions on the [Install the Microsoft ODBC driver for SQL Server (Linux)](../../connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server.md).  Make sure to also install the `unixodbc-dev` package. It's used by the `pecl` command to install the PHP drivers.

You may also need to generate the correct locale to get PHP output to display correctly in a browser. For example, for the en_US UTF-8 locale, run the following commands:

```bash
sudo su
sed -i 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/g' /etc/locale.gen
locale-gen
```

You may need to add `/usr/sbin` to your `$PATH`, as the `locale-gen` executable is located there.

### Step 3. Install the PHP drivers for Microsoft SQL Server (Debian)

```bash
sudo pecl install sqlsrv
sudo pecl install pdo_sqlsrv
sudo su
printf "; priority=20\nextension=sqlsrv.so\n" > /etc/php/8.1/mods-available/sqlsrv.ini
printf "; priority=30\nextension=pdo_sqlsrv.so\n" > /etc/php/8.1/mods-available/pdo_sqlsrv.ini
exit
sudo phpenmod -v 8.1 sqlsrv pdo_sqlsrv
```

If there is only one PHP version in the system, then the last step can be simplified to `phpenmod sqlsrv pdo_sqlsrv`. As with `locale-gen`, `phpenmod` is located in `/usr/sbin` so you may need to add this directory to your `$PATH`.

### Step 4. Install Apache and configure driver loading (Debian)

```bash
sudo su
apt-get install libapache2-mod-php8.1 apache2
a2dismod mpm_event
a2enmod mpm_prefork
a2enmod php8.1
```

### Step 5. Restart Apache and test the sample script (Debian)

```bash
sudo service apache2 restart
```

To test your installation, see [Testing your installation](#testing-your-installation) at the end of this document.

## Installing on SUSE


> [!NOTE]
> In the following instructions, replace `<SuseVersion>` with your version of SUSE - if you are using SUSE Linux Enterprise Server 15, it will be SLE_15_SP3 or SLE_15_SP4 (or above). For SUSE 12, use SLE_12_SP5 (or above). Not all versions of PHP are available for all versions of SUSE Linux - please refer to `http://download.opensuse.org/repositories/devel:/languages:/php` to see which versions of SUSE have the default version PHP available, or check `http://download.opensuse.org/repositories/devel:/languages:/php:/` to see which other versions of PHP are available for which versions of SUSE.

> [!NOTE]
> Packages for PHP 7.4 or above are not available for SUSE 12, as of today.

### Step 1. Install PHP (SUSE)

```bash
sudo su
zypper -n ar -f https://download.opensuse.org/repositories/devel:languages:php/<SuseVersion>/devel:languages:php.repo
zypper --gpg-auto-import-keys refresh
zypper -n install php8 php8-pdo php8-devel php8-openssl
```

### Step 2. Install prerequisites (SUSE)

Install the ODBC driver for SUSE by following the instructions on the [Install the Microsoft ODBC driver for SQL Server (Linux)](../../connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server.md). Make sure to also install the `unixodbc-dev` package. It's used by the `pecl` command to install the PHP drivers.

### Step 3. Install the PHP drivers for Microsoft SQL Server (SUSE)

```bash
sudo pecl install sqlsrv
sudo pecl install pdo_sqlsrv
sudo su
echo extension=pdo_sqlsrv.so >> `php --ini | grep "Scan for additional .ini files" | sed -e "s|.*:\s*||"`/pdo_sqlsrv.ini
echo extension=sqlsrv.so >> `php --ini | grep "Scan for additional .ini files" | sed -e "s|.*:\s*||"`/sqlsrv.ini
exit
```

### Step 4. Install Apache and configure driver loading (SUSE)

```bash
sudo su
zypper install apache2 apache2-mod_php8
a2enmod php8
echo "extension=sqlsrv.so" >> /etc/php8/apache2/php.ini
echo "extension=pdo_sqlsrv.so" >> /etc/php8/apache2/php.ini
exit
```

### Step 5. Restart Apache and test the sample script (SUSE)

```bash
sudo systemctl restart apache2
```

To test your installation, see [Testing your installation](#testing-your-installation) at the end of this document.

## Installing on Alpine


> [!NOTE]
> PHP 8.1 or above may be available from testing or edge repositories for Alpine. You can instead compile PHP from source.

### Step 1. Install PHP (Alpine)

PHP packages for Alpine can be found in the `edge/community` repository. Check [Enable Community Repository](https://wiki.alpinelinux.org/wiki/Enable_Community_Repository) on their WIKI page. Add the following line to `/etc/apk/repositories`, replacing `<mirror>` with the URL of an Alpine repository mirror:

```bash
http://<mirror>/alpine/edge/community
```

Then run:

```bash
sudo su
apk update
# Note: The php*-pdo package is required only for the PDO_SQLSRV driver
# For PHP 7.*
apk add php7 php7-dev php7-pear php7-pdo php7-openssl autoconf make g++
# For PHP 8.*
apk add php8 php8-dev php8-pear php8-pdo php8-openssl autoconf make g++
# The following symbolic links are optional but useful
ln -s /usr/bin/php8 /usr/bin/php
ln -s /usr/bin/phpize8 /usr/bin/phpize
ln -s /usr/bin/pecl8 /usr/bin/pecl
ln -s /usr/bin/php-config8 /usr/bin/php-config
```

### Step 2. Install prerequisites (Alpine)

Install the ODBC driver for Alpine by following the instructions on the [Install the Microsoft ODBC driver for SQL Server (Linux)](../../connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server.md).  Make sure to also install the `unixodbc-dev` package (`sudo apk add unixodbc-dev`). It's used by the `pecl` command to install the PHP drivers.

### Step 3. Install the PHP drivers for Microsoft SQL Server (Alpine)

```bash
sudo pecl install sqlsrv
sudo pecl install pdo_sqlsrv
sudo su
echo extension=pdo_sqlsrv.so >> `php --ini | grep "Scan for additional .ini files" | sed -e "s|.*:\s*||"`/10_pdo_sqlsrv.ini
echo extension=sqlsrv.so >> `php --ini | grep "Scan for additional .ini files" | sed -e "s|.*:\s*||"`/20_sqlsrv.ini
```

### Step 4. Install Apache and configure driver loading (Alpine)

```bash
# For PHP 7.*
sudo apk add php7-apache2 apache2
# For PHP 8.*
sudo apk add php8-apache2 apache2
```

### Step 5. Restart Apache and test the sample script (Alpine)

```bash
sudo rc-service apache2 restart
```

To test your installation, see [Testing your installation](#testing-your-installation) at the end of this document.

## Installing on macOS


If you do not already have it, install brew as follows:

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

> [!NOTE]
> To install PHP 8.0 or 8.2, replace php@8.1 with php@8.0 or php@8.2 respectively in the following commands.

### Step 1. Install PHP (macOS)

```bash
brew tap
brew tap homebrew/core
brew install php@8.1
```

PHP should now be in your path. Run `php -v` to verify that you are running the correct version of PHP. If PHP is not in your path or it is not the correct version, run the following commands:

```bash
brew link --force --overwrite php@8.1
```

If using Apple M1 ARM64, you might need to set the path:
```bash
export PATH="/opt/homebrew/bin:$PATH"
```

### Step 2. Install prerequisites (macOS)

Install the ODBC driver for macOS by following the instructions on the [Install the Microsoft ODBC driver for SQL Server (macOS)](../../connect/odbc/linux-mac/install-microsoft-odbc-driver-sql-server-macos.md).

> [!NOTE]
> If using Apple M1 ARM64 hardware, please install Microsoft ODBC driver 17.8+ directly without using the emulator Rosetta 2.

In addition, you may need to install the GNU make tools:

```bash
brew install autoconf automake libtool
```

### Step 3. Install the PHP drivers for Microsoft SQL Server (macOS)

```bash
sudo pecl install sqlsrv
sudo pecl install pdo_sqlsrv
```

If using Apple M1 ARM64, do the following instead:
```bash
sudo CXXFLAGS="-I/opt/homebrew/opt/unixodbc/include/" LDFLAGS="-L/opt/homebrew/lib/" pecl install sqlsrv
sudo CXXFLAGS="-I/opt/homebrew/opt/unixodbc/include/" LDFLAGS="-L/opt/homebrew/lib/" pecl install pdo_sqlsrv
```

### Step 4. Install Apache and configure driver loading (macOS)

> [!NOTE]
> The latest macOS 11.0 Big Sur comes with Apache 2.4 pre-installed, but Apple has also removed some required scripts. The solution is to install Apache 2.4 via Homebrew and then configure it, but this is out of scope for this installation guide, so please check Apache or Homebrew for detailed instructions.

```bash
brew install apache2
```

To find the Apache configuration file, `httpd.conf`, for your Apache installation, run:

```bash
/usr/local/bin/apachectl -V | grep SERVER_CONFIG_FILE
```

The following commands append the required configuration to `httpd.conf`. Be sure to substitute the path returned by the preceding command in place of `/usr/local/etc/httpd/httpd.conf`:

```bash
echo "LoadModule php7_module /usr/local/opt/php@8.1/lib/httpd/modules/libphp7.so" >> /usr/local/etc/httpd/httpd.conf
(echo "<FilesMatch .php$>"; echo "SetHandler application/x-httpd-php"; echo "</FilesMatch>";) >> /usr/local/etc/httpd/httpd.conf
```

### Step 5. Restart Apache and test the sample script (macOS)

```bash
sudo apachectl restart
```

To test your installation, see [Testing your installation](#testing-your-installation) at the end of this document.

## Testing Your Installation

To test this sample script, create a file called testsql.php in your system's document root. This path is `/var/www/html/` on Ubuntu, Debian, and Red Hat, `/srv/www/htdocs` on SUSE, `/var/www/localhost/htdocs` on Alpine, or `/usr/local/var/www` on macOS. Copy the following script to it, replacing the server, database, username, and password as appropriate.

### SQLSRV example

```php
<?php
$serverName = "yourServername";
$connectionOptions = array(
    "database" => "yourDatabase",
    "uid" => "yourUsername",
    "pwd" => "yourPassword"
);

function exception_handler($exception) {
    echo "<h1>Failure</h1>";
    echo "Uncaught exception: " , $exception->getMessage();
    echo "<h1>PHP Info for troubleshooting</h1>";
    phpinfo();
}

set_exception_handler('exception_handler');

// Establishes the connection
$conn = sqlsrv_connect($serverName, $connectionOptions);
if ($conn === false) {
    die(formatErrors(sqlsrv_errors()));
}

// Select Query
$tsql = "SELECT @@Version AS SQL_VERSION";

// Executes the query
$stmt = sqlsrv_query($conn, $tsql);

// Error handling
if ($stmt === false) {
    die(formatErrors(sqlsrv_errors()));
}
?>

<h1> Success Results : </h1>

<?php
while ($row = sqlsrv_fetch_array($stmt, SQLSRV_FETCH_ASSOC)) {
    echo $row['SQL_VERSION'] . PHP_EOL;
}

sqlsrv_free_stmt($stmt);
sqlsrv_close($conn);

function formatErrors($errors)
{
    // Display errors
    echo "<h1>SQL Error:</h1>";
    echo "Error information: <br/>";
    foreach ($errors as $error) {
        echo "SQLSTATE: ". $error['SQLSTATE'] . "<br/>";
        echo "Code: ". $error['code'] . "<br/>";
        echo "Message: ". $error['message'] . "<br/>";
    }
}
?>
```

### PDO_SQLSRV example

```php
<?php
try {
    $serverName = "yourServername";
    $databaseName = "yourDatabase";
    $uid = "yourUsername";
    $pwd = "yourPassword";
    
    $conn = new PDO("sqlsrv:server = $serverName; Database = $databaseName;", $uid, $pwd);

    // Select Query
    $tsql = "SELECT @@Version AS SQL_VERSION";

    // Executes the query
    $stmt = $conn->query($tsql);
} catch (PDOException $exception1) {
    echo "<h1>Caught PDO exception:</h1>";
    echo $exception1->getMessage() . PHP_EOL;
    echo "<h1>PHP Info for troubleshooting</h1>";
    phpinfo();
}

?>

<h1> Success Results : </h1>

<?php
try {
    while ($row = $stmt->fetch(PDO::FETCH_ASSOC)) {
        echo $row['SQL_VERSION'] . PHP_EOL;
    }
} catch (PDOException $exception2) {
    // Display errors
    echo "<h1>Caught PDO exception:</h1>";
    echo $exception2->getMessage() . PHP_EOL;
}

unset($stmt);
unset($conn);
?>
```

Point your browser to `https://localhost/testsql.php` (`https://localhost:8080/testsql.php` on macOS). You should now be able to connect to your SQL Server/Azure SQL database. If you don't see a success message showing SQL version information, you can do some basic troubleshooting by running the script from the command line:

```bash
php testsql.php
```

If running from the command line is successful but nothing shows in your browser, check the [Apache log files](https://linuxize.com/post/apache-log-files/#location-of-the-log-files). For more help, see [Support resources](support-resources-for-the-php-sql-driver.md) for places to go.

## See Also

[Getting Started with the Microsoft Drivers for PHP for SQL Server](../../connect/php/getting-started-with-the-php-sql-driver.md)

[Loading the Microsoft Drivers for PHP for SQL Server](../../connect/php/loading-the-php-sql-driver.md)

[System Requirements for the Microsoft Drivers for PHP for SQL Server](../../connect/php/system-requirements-for-the-php-sql-driver.md)
