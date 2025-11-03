# Use the official PHP Apache image as the base image
FROM php:8.2-apache

# Install PHP extensions and dependencies
RUN docker-php-ext-install mysqli pdo pdo_mysql

# Enable Apache modules
RUN a2enmod rewrite

# Set working directory
WORKDIR /var/www/html


