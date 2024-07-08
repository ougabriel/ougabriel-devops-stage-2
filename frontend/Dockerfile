
# FROM node:20

# # Set working directory
# WORKDIR /app

# # Install dependencies
# COPY package*.json ./
# RUN npm install

# # Copy the rest of the application
# COPY . .

# # Build the application
# CMD npm run dev -- --host 0.0.0.0

# # Use a smaller image for serving the build
# FROM nginx:alpine
# COPY --from=dev /app/dist /usr/share/nginx/html

# # Expose port 80
# EXPOSE 80



# Stage 1: Build the application
FROM node:20 AS build

# Set working directory
WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm install

# Copy the rest of the application
COPY . .

# Build the application
RUN npm run build

# Stage 2: Serve the build with Nginx
FROM nginx:alpine

# Copy the build from the previous stage
COPY --from=build /app/dist /usr/share/nginx/html

# Expose port 80
EXPOSE 4173

CMD ["npm","run","preview","--","--host", "0.0.0.0"]

