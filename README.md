
---

# Docker Container Creation and Setup Report

## 1. Setting Up the Docker Container

### 1.1 Dockerfile
The Dockerfile was configured to create an environment with PHP 7.2, MySQL 5.7, Composer, Node.js, and other required components. Ensure all commands are included in the Dockerfile for a smooth setup.

Here is the Dockerfile used:

```docker 
FROM ubuntu:20.04

ARG DEBIAN_FRONTEND=noninteractive

# Update package lists and install dependencies
RUN apt-get update -y && \
    apt-get install -y \
    software-properties-common \
    curl \
    wget \
    git \
    nginx \
    lsb-release

# Add the PHP repository for PHP 7.2
RUN add-apt-repository ppa:ondrej/php && \
    apt-get update -y

# Install PHP 7.2 and extensions
RUN apt-get install -y \
    php7.2 \
    php7.2-fpm \
    php7.2-bcmath \
    php7.2-xml \
    php7.2-mysql \
    php7.2-zip \
    php7.2-intl \
    php7.2-ldap \
    php7.2-gd \
    php7.2-cli \
    php7.2-bz2 \
    php7.2-curl \
    php7.2-mbstring \
    php7.2-pgsql \
    php7.2-soap \
    php7.2-cgi


RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer --version=2.2.24


# install NVM (node Version Manager)
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash && \
    export NVM_DIR="$HOME/.nvm" && \
    [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# install node.js using NVM and set default node.js version
RUN export NVM_DIR="$HOME/.nvm" && \
    [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" && \
    nvm install 10 && \
    nvm use 10 && \
    nvm alias default 10 && \
    npm install -g npm@6

# These are the commands I ran on my system, and I successfully installed MySQL 5.7. 
# Please note that some of these commands may require user input.
# this below command has isuue with errors, so its better to run it manually after created the container 

#RUN wget https://dev.mysql.com/get/mysql-apt-config_0.8.12-1_all.deb && \
#    echo "mysql-apt-config mysql-apt-config/select-server select mysql-5.7" | debconf-set-selections && \
#    dpkg -i mysql-apt-config_0.8.12-1_all.deb && \
#    apt-get update -y && \
#    apt-cache policy mysql-server
#    apt update
#    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 467B942D3A79BD29 && \
#    apt update
#    apt-get update --allow-insecure-repositories && \
#    apt-get install -f -y mysql-client=5.7* mysql-community-server=5.7* mysql-server=5.7*


RUN rm -rf /var/www/html/*


# git remote repo set up. It may require a username and Personal Access Token (PAT) for access repo.

RUN git config --global http.postBuffer 524288000

RUN git clone https://<github-PAT-Token>@github.com/abc/development-erp.git /var/www/html/development-erp


RUN chown -R www-data:www-data /var/www

# run this below command if want to change mysql user 

#    RUN service mysql start && \
#    mysql -e "CREATE DATABASE IF NOT EXISTS laravel;" && \
#    mysql -e "CREATE USER 'laravel'@'localhost' IDENTIFIED BY 'laravel';" && \
#    mysql -e "GRANT ALL PRIVILEGES ON laravel.* TO 'laravel'@'localhost';" && \
#    mysql -e "FLUSH PRIVILEGES;"


EXPOSE 80 3306

CMD ["bash", "-c", "service php7.2-fpm start && nginx -g 'daemon off;'"]


```

## 2. Running the Docker Container

### 2.1 Build the Docker Image
```bash
docker build -t laravel-app .
```

### 2.2 Run the Docker Container
```bash
docker run -d --name laravel-container -p 8080:80 -p 3306:3306 laravel-app
```

## 3. Post-Container Setup

### 3.1 Navigate to Project Directory
```bash
cd /var/www/html/development-erp/
```

### 3.2 Check Files and Permissions
```bash
ls -la
```

### 3.3 Manual MySQL Installation
Due to Docker build issues, these commands were run manually after container creation:

```bash
# Download MySQL APT configuration
wget https://dev.mysql.com/get/mysql-apt-config_0.8.12-1_all.deb 

# Configure MySQL server version
echo "mysql-apt-config mysql-apt-config/select-server select mysql-5.7" | debconf-set-selections 

# Install MySQL APT configuration
dpkg -i mysql-apt-config_0.8.12-1_all.deb 

# Update package lists
apt-get update -y 

# Check MySQL server policy
apt-cache policy mysql-server

# Update package lists again
apt update

# Add MySQL APT repository key
apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 467B942D3A79BD29 

# Update package lists
apt update

# Allow insecure repositories and install MySQL
apt-get update --allow-insecure-repositories 
apt-get install -f -y mysql-client=5.7* mysql-community-server=5.7* mysql-server=5.7*
```

### 3.4 Check MySQL Installation
```bash
# Verify MySQL version
mysql --version

# Restart MySQL service
service mysql restart
```

### 3.5 Import SQL Database
```bash
# Import SQL database
mysql -u root -p dev_erp < DB/dev_erp_db_backup-26-08-2024.sql
```

### 3.6 Remove Previous Git Repository
```bash
# List files and remove previous Git repository
ls -la
rm -rf .git
ls -la
```

### 3.7 Initialize New Git Repository
```bash
# Initialize a new Git repository and set global configuration
git init
git config --global user.email "user@gmail.com"
git config --global user.name "username"
```

### 3.8 Install Project Dependencies
```bash
# Update Composer dependencies and run Artisan commands
composer update
php artisan route:list
php artisan passport:install
```

## 4. Issue Resolution

### 4.1 Issue: `Class App\Http\Controllers\IntercompanysaleController` Does Not Exist
- **Solution**: Comment out `IntercompanysaleController` in `api.php`.

### 4.2 Issue: `ReflectionException: Class App\Http\Controllers\PaymentAccountController` Does Not Exist
- **Solution**: Comment out `PaymentAccountController` in `web.php` at line 68.

### 4.3 Issue: `ReflectionException: Class App\Http\Controllers\CategoryController` Does Not Exist
- **Solution**: Comment out `CategoryController` in `web.php` at line 774.

### 4.4 Issue: Update `AppServiceProvider`
- **Solution**: Replace the code from lines 53 to 108 in `AppServiceProvider` with the following code:

```php
View::composer(
    ['*'],
    function ($view) {
        $enabled_modules = !empty(session('business.enabled_modules')) ? session('business.enabled_modules') : [];
        $__is_pusher_enabled = isPusherEnabled();

        if (!Auth::check()) {
            $__is_pusher_enabled = false;
        }

        $authUser = auth()->check() ? auth()->user()->id : null;
        $view->with('enabled_modules', $enabled_modules);
        $view->with('__is_pusher_enabled', $__is_pusher_enabled);

        $business_id = request()->session()->get('user.business_id') ?? 1;
        $business = \App\Business::where('id', $business_id)->first();

        if ($business) {
            $checkFinancialYear = \App\UserFinancialYear::where('business_id', $business_id)
                ->where('user_id', $authUser)
                ->first();

            if (!request()->session()->get('financial_year_sel')) {
                $cyear = \Carbon\Carbon::now()->year;
                $cmonth = \Carbon\Carbon::now()->month;

                if ($business->fy_start_month > $cmonth) {
                    $cyear = $cyear - 1;
                }

                $startDate = date('Y-m-d', strtotime('1-' . $business->fy_start_month . '-' . $cyear));
                $endDate = date('Y-m-d', strtotime('t-' . ($business->fy_start_month - 1) . '-' . ($cyear + 1)));
            } else {
                if ($checkFinancialYear) {
                    $startDate = date('Y-m-d', strtotime('1-' . $checkFinancialYear->fy_start_month . '-' . $checkFinancialYear->financial_year_start));
                    $endDate = date('Y-m-d', strtotime('t-' . ($checkFinancialYear->fy_start_month - 1) . '-' . $checkFinancialYear->financial_year_end));
                } else {
                    $cyear = \Carbon\Carbon::now()->year;
                    $cmonth = \Carbon\Carbon::now()->month;

                    if ($business->fy_start_month > $cmonth) {
                        $cyear = $cyear - 1;
                    }

                    $startDate = date('Y-m-d', strtotime('1-' . $business->fy_start_month . '-' . $cyear));
                    $endDate = date('Y-m-d', strtotime('t-' . ($business->fy_start_month - 1) . '-' . ($cyear + 1)));
                }
            }

            $view->with('startDate', $startDate);
            $view->with('endDate', $endDate);
        } else {
            // Handle the case where the business object is null
            $view->with('startDate', null);
            $view->with('endDate', null);
        }
    }
);
```

---
