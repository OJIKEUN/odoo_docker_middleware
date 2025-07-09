FROM php:7.4-fpm

# Install dependencies
RUN apt-get update && apt-get install -y \
    unzip curl libaio1 libaio-dev build-essential \
    libpng-dev libjpeg-dev libfreetype6-dev apt-transport-https \
    ca-certificates gnupg2 lsb-release && \
    rm -rf /var/lib/apt/lists/*

# Install Oracle Instant Client
WORKDIR /opt/oracle
COPY oracle-client/instantclient-basic-linux.x64-19.26.0.0.0dbru.zip .
COPY oracle-client/instantclient-sdk-linux.x64-19.26.0.0.0dbru.zip .

RUN unzip -o instantclient-basic-linux.x64-19.26.0.0.0dbru.zip && \
    unzip -o instantclient-sdk-linux.x64-19.26.0.0.0dbru.zip && \
    rm -f *.zip

# Set environment variables
ENV ORACLE_HOME=/opt/oracle/instantclient_19_26
ENV LD_LIBRARY_PATH=$ORACLE_HOME
ENV PATH="$ORACLE_HOME:$PATH"
ENV TNS_ADMIN=$ORACLE_HOME

# Konfigurasi symlink untuk library
RUN ln -s $ORACLE_HOME/libclntsh.so.19.1 /usr/lib/libclntsh.so && \
    echo "/opt/oracle/instantclient_19_26" > /etc/ld.so.conf.d/oracle-instantclient.conf && ldconfig

# Install OCI8 versi 2.2.0 untuk PHP 7.4
RUN docker-php-source extract && \
    echo "instantclient,/opt/oracle/instantclient_19_26" | pecl install oci8-2.2.0 && \
    docker-php-ext-enable oci8 && \
    docker-php-source delete
    
# Install PHP extensions (PDO, PDO MySQL, GD)
RUN docker-php-ext-install pdo pdo_mysql mysqli gd

# Install Composer
WORKDIR /usr/local/bin
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \
    php composer-setup.php && \
    php -r "unlink('composer-setup.php');" && \
    mv composer.phar /usr/local/bin/composer

# Set working directory
WORKDIR /var/www/html

# Set permission for Laravel storage & cache folders
# ARG PROJECT_FOLDER
# RUN chown -R www-data:www-data /var/www/html/${PROJECT_FOLDER}/storage /var/www/html/${PROJECT_FOLDER}/bootstrap/cache && \
#     chmod -R 775 /var/www/html/${PROJECT_FOLDER}/storage /var/www/html/${PROJECT_FOLDER}/bootstrap/cache

CMD ["php-fpm"]
