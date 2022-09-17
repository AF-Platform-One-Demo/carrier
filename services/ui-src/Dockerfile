# ==============================================================================
# => Dev container
FROM node:14.18.3 as dev

# Set working directory
WORKDIR /app

# Start up sequence for the dev container
CMD ["/bin/bash", "-c", "yarn install && ./env.sh && cp ./env-config.js ./public/ && yarn run start"]


# ==============================================================================
# => Build container
FROM dev as builder

# Copy package.json and package-lock.json
COPY package.json yarn.lock ./

# Install modules from the package-lock.json
RUN yarn install --frozen-lockfile

# Copy source code
COPY . .

# Create static build
RUN yarn run build


# ==============================================================================
# => Run container
FROM nginx:1.22.0-alpine as runner

# Copy nginx config
RUN rm -rf /etc/nginx/conf.d
COPY conf /etc/nginx

# Copy static build from the builder container
COPY --from=builder /app/build /usr/share/nginx/html/

# Expose port 80 for nginx
EXPOSE 80

# Copy .env file and shell script to container; this sets runtime variables
WORKDIR /usr/share/nginx/html
COPY ./env.sh .
COPY .env .

# Add bash to image, so env.sh can be run
RUN apk add --no-cache bash

# Make env.sh executable
RUN chmod +x env.sh

# Start Nginx server; env.sh is run first, setting runtime variables on boot
CMD ["/bin/bash", "-c", "/usr/share/nginx/html/env.sh && nginx -g \"daemon off;\""]
