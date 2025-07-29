# Dockerfile
FROM nginx:alpine

# Create non-root user with UID 1000
RUN addgroup -g 1000 appgroup && adduser -u 1000 -G appgroup -S appuser

# Copy static file into nginx HTML directory
COPY index.html /usr/share/nginx/html/index.html

# Adjust permissions (optional but good practice)
RUN chown -R appuser:appgroup /usr/share/nginx/html

# Switch to non-root user
USER appuser

EXPOSE 80

# Run nginx in foreground
CMD ["nginx", "-g", "daemon off;"]
