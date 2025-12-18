# Minimal PHP service using Apache
FROM php:8.2-apache

# Copy the application code
COPY index.php /var/www/html/index.php

# Expose HTTP port
EXPOSE 80

# Healthcheck (optional but helpful)
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -fsS http://localhost/ || exit 1
