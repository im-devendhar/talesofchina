# Use official lightweight Nginx image
FROM nginx:alpine

# Set working directory inside container
WORKDIR /usr/share/nginx/html

# Remove default nginx static assets
RUN rm -rf ./*

# Copy your static website files into the container
COPY index.html ./
# Uncomment and copy additional directories if needed
# COPY css/ ./css/
# COPY js/ ./js/
# COPY images/ ./images/

# Copy custom Nginx configuration if you have one
# COPY nginx.conf /etc/nginx/nginx.conf

# Expose port 80 for HTTP traffic
EXPOSE 80

# Use the default Nginx entrypoint
CMD ["nginx", "-g", "daemon off;"]
