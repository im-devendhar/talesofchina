# Use official lightweight Nginx image
FROM nginx:alpine

# Set working directory inside container
WORKDIR /usr/share/nginx/html

# Remove default nginx static assets
RUN rm -rf ./*

# Copy your static website files into the container
COPY index.html ./
# If you have other files like CSS, JS, images, copy them too
# COPY css/ ./css/
# COPY js/ ./js/
# COPY images/ ./images/

# Expose port 80 for HTTP traffic
EXPOSE 80

# Nginx automatically serves /usr/share/nginx/html
