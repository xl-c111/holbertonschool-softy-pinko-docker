# Use the latest nginx image
FROM nginx:latest

# Copy frontend built files into nginx html repository
COPY softy-pinko-front-end /var/www/html/softy-pinko-front-end

# Copy custom nginx config into container
COPY softy-pinko-front-end.conf /etc/nginx/conf.d/default.conf

# Expose port
EXPOSE 9000


# The logic of Dockerfile
# base image(Nignx): provides the web server
# site files are copied into the container at a path you control
# Nignx config tells Nignx inside the container which port to listen to
# Expose tells Docker the container uses port 9000