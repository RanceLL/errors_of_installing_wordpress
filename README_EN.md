
I tried to install Wordpress with LAMP in AWS and met tons of errors and bugs.
Here I'm going to make a summary of these errors.
Hope it could be helpful for you.

#ENVIRONMENT
* remote server: AWS Ubuntu 14.04
* local OS: Mac OS(Windows would also be OK)＊
* connect to the internet

＊ Since we need to use PuTTY to connect to the remote server and as far as I know, it's difficult to copy&paste in PuTTY, so better using MacOS.

#SUMMARY OF ERRORS 

##Error_1
Type command lines like this

    sudo apt-get install php5
and get 

    E: Package 'php4' has no installation candidate
##Analyzing&Solution
Check the AMI's version first. (please install the instance of Ubuntu 14.04, otherwise, you have to install php7 instead, which may lead to some errors that hard to solve)

##Error_2
Type command lines like this

    sudo apt-get install php5
and get
    perl: warning: Setting locale failed.
    perl: warning: Please check that your locale settings:
       LANGUAGE = "ru_RU.UTF-8",
       LC_ALL = “",
        ・・・・・・・・
##Analyzing&Solution
There may be something wrong with the language setting(I am not pretty sure about why this error happened but command lines come next may be helpful).

    sudo locale-gen en_US en_US.UTF-8
    sudo dpkg-reconfigure locales
    export LC_ALL=en_US.UTF-8
    sudo apt-get install --reinstall language-pack-en-base
Moreover, you may try to add codes directly into the /etc/environment, like this

    nano /etc/environment
    (addingCode:   LC_ALL=“en_GB.utf8"   to /etc/environment and    rebooting.)

##Error_3
Do not know the username and password in "YOURDOMAIN/phpmyadmin" login page.
##Analyzing&Solution
Well, let's start from resetting MySQL account.
    mysql -u root -p
    CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
    GRANT ALL ON wordpress.* TO 'root'@'localhost' IDENTIFIED BY '12345678'';
    FLUSH PRIVILEGES;
    EXIT;
Try to login with root(username) / 12345678(password).

##Error_4
after installing wordpress, still can not open the "Domain/wp-admin" page.
##Analyzing&Solution
May be we need to change some settings of "wp-config.php".
Before that, better to update the database by 

    mysql -u root -p
    CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
    GRANT ALL ON wordpress.* TO 'newuser'@'localhost' IDENTIFIED BY 'password';
    FLUSH PRIVILEGES;
    EXIT;
Then, 

    nano /var/www/wordpress/wp-config.php
edit the wp-config.php like this

    define(‘DB_NAME’, ‘wordpress‘);
    define(‘DB_USER’, ‘newuser‘);
    define(‘DB_HOST’, ‘localhost‘);
After that, get your secret key from wordpress. 
https://api.wordpress.org/secret-key/1.1/salt/
replace fill them into wp-config.php

    define('AUTH_KEY', daskgjlqkasdfasdfasdfwehnoisjd3');
    define('SECURE_AUTH_KEY',  'tasdfasd-qegqwadgasu,D');
    define('LOGGED_IN_KEY',    'asdfqghjtykugfgherteFc');
    define('NONCE_KEY',        '&gqergo0pijhoijhafqwfB');
    define('AUTH_SALT',        '+j12098u8htgnehg408u9X');
    define('SECURE_AUTH_SALT', 'X`nM$+j4{(~t.A'&%T:agh');
    define('NONCE_SALT',     %'&()YG%$&'YHBJJNIJJKHa4?');

    
##Error_5
Be asked for username and password to install plugins.
##Analyzing&Solution
It could be the reason that apache can not access folders and files.
Try command lines like this to solve it.

    sudo chown www-data:www-data -R /var/www/
    sudo chown -R apache:apache /var/www
Then, add codes into the wp-config.php

      define('FS_METHOD','direct’);


##Error_6
The theme could not be rendered well.
##Analyzing&Solution
goto [YOURDOMAIN/wp-admin] → [Setting] → [General], replace those long domain address, such as "ec2-12-345-678-901.ap-southeast-1.compute.amazonaws.com"with your url like www.AAAAA.com
