FROM kitware/cdash:latest

LABEL maintainer="Kitware, Inc. <cdash@public.kitware.com>"

LABEL com.circleci.preserve-entrypoint=true

ARG CDASH_DATABASE_HOST=localhost

# Creates the kitware home directory with directory for cdash
WORKDIR /var/www/cdash

# Copy CDash (current folder) into /home/kitware/cdash to set its owner
# to www-data
ADD --chown=www-data:www-data . ./

# Create links in the public folder to avoid Laravel handling requests
# for assets
#RUN ln -s /var/www/cdash/app/cdash/public/build public/build \
# && ln -s /var/www/cdash/app/cdash/public/css public/css \
# && ln -s /var/www/cdash/app/cdash/public/fonts public/fonts \
# && ln -s /var/www/cdash/app/cdash/public/img public/img \
# && ln -s /var/www/cdash/app/cdash/public/js public/js \
# && ln -s /var/www/cdash/app/cdash/public/views public/views

WORKDIR /var/www/cdash/app/cdash

RUN apt-get install -y cmake \
 # it's not obvious here but cmake runs  composer, npm and gulp
 && pecl install xdebug                        \
 && docker-php-ext-enable xdebug            \
 && cp php.ini /usr/local/etc/php/conf.d/cdash.ini
 && mkdir _build && cd _build \
 && cmake \
  -DCDASH_DB_HOST=$CDASH_DATABASE_HOST \
  -DCDASH_DIR_NAME= \
  -DCDASH_TESTING_URL='http://localhost/' \
  -DCDASH_SERVER='localhost' \
  ..

# Install Laravel dependencies with composer and npm, configure Laravel
WORKDIR /var/www/cdash

RUN composer install --no-interaction --no-progress --prefer-dist \
 && npm install                                                             \
 # copy the docker specific environment file from the docker directory
 && cp docker/.env.docker .env                                              \
 && php artisan key:generate                                                \
 # copy the docker specific protractor configuration file for js testing
 && cp docker/conf.js app/cdash/_build/tests/js/e2e_tests/conf.js         \
 && chown -R www-data:www-data .

# Configure Apache
RUN cp /var/www/cdash/docker/cdash-site.conf /etc/apache2/sites-available/cdash-site.conf
# Remove default site, add cdash-site, enable mod_rewrite, enable php7
RUN a2dissite 000-default  \
 && a2ensite cdash-site    \
 && a2enmod rewrite        \
 && a2enmod php7

EXPOSE 80

ENTRYPOINT ["ctest", "/var/www/cdash/app/cdash/_build"]
