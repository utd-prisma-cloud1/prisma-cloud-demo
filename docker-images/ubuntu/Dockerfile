# Pull the minimal Ubuntu image
FROM us-central1-docker.pkg.dev/panw-utd-public-cloud/utd-demo-images/utd-cnsp/ubuntu:malware

# Install Nginx
RUN apt-get -qq -y update && apt-get -qq -y install nginx

# Copy the Nginx config
COPY default /etc/nginx/sites-available/default

# Expose the port for access
EXPOSE 80/tcp

# Run the Nginx server
CMD /opt/entrypoint.sh && /usr/sbin/nginx -g daemon off
