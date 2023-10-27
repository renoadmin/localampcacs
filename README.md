# Docker Demo Project

## Dockerfile Explanation

This README provides a step-by-step explanation of the Dockerfile provided. The Dockerfile uses multi-stage builds to create a container for a Node.js application, separating the build environment from the production environment.

### Stage 1: Base (Build)

1. **Base Image Selection**:
   - `FROM node:18.13.0 AS base`: This line specifies the base image for the first stage of the build. It uses Node.js version 18.13.0 as the starting point and names this stage "base."

2. **Working Directory Setup**:
   - `WORKDIR /app`: Sets the working directory within the container to `/app`.

3. **Dependency Installation**:
   - `COPY package*.json ./`: Copies the `package.json` and `package-lock.json` files from your host machine to the `/app` directory in the container.
   - `RUN npm install`: Installs the application's Node.js dependencies and their dependencies using npm.

4. **Prisma Setup**:
   - `COPY prisma/schema.prisma ./prisma/`: Copies the Prisma schema file to the `/app/prisma/` directory.
   - `RUN npm run prisma:generate`: Generates the Prisma client based on the schema file.

5. **Codebase Copy**:
   - `COPY . .`: Copies the entire codebase from your host machine into the `/app` directory in the container.

6. **Application Build**:
   - `RUN npm run build`: Builds the application using `npm run build`.

### Stage 2: Production (Runtime)

This stage starts with a new base image (`node:18.13.0-slim`) and is intended for the production environment.

1. **Build-time Variables**:
   - `ARG` lines define build-time variables that can be set when building the Docker image.

2. **Temporary Workaround**:
   - `[temporary] work around to be able to run prisma`: This line is a temporary workaround to install the openssl package. It seems to be related to running Prisma.

3. **Working Directory Setup**:
   - `WORKDIR /app`: Sets the working directory to `/app` again.

4. **User and Group Creation**:
   - `RUN groupadd --gid ${gid} ${user}`: Creates a group with the specified UID and GID.
   - `RUN useradd --uid ${uid} --gid ${gid} -m ${user}`: Creates a user with the specified UID and GID, with the home directory at `/app`.

5. **Copy Artifacts**:
   - The `COPY` commands bring over artifacts from the previous build stage. These commands copy:
     - `/app/node_modules/` and `/app/package.json` for Node.js dependencies.
     - `/app/dist` for the built application.
     - `/app/prisma` for Prisma files.
     - `/app/scripts` for scripts.
     - `/app/src` for source code.
     - `/app/tsconfig*` for TypeScript configuration files.

6. **Ownership Change**:
   - `RUN chown -R ${uid}:${gid} /app/`: Changes the ownership of the workspace directory to the non-privileged user created earlier.

7. **Production Dependency Installation**:
   - `RUN npm install --production`: Installs production dependencies only, excluding development dependencies.

8. **User Switch**:
   - `USER ${user}`: Sets the user to the non-privileged user created earlier.

9. **Environment Variables and Port Exposition**:
   - `ENV PORT=3000`: Sets an environment variable `PORT` with a value of 3000.
   - `EXPOSE ${PORT}`: Exposes port 3000 within the Docker container.

10. **Container Start Command**:
    - `CMD [ "node", "./dist/main.js" ]`: Specifies the command to start the Node.js application when the container is run. It runs `node` on the `main.js` file located in the `/app/dist` directory.

This Dockerfile separates the build environment from the production environment, optimizes the Docker image size, and sets up the application for production use.

# Node.js Application Dockerization with Multi-Stage Build

This Dockerfile is designed to containerize a Node.js application using multi-stage builds. It separates the build environment from the production environment to optimize the image size and prepare the application for production use.

## Stage 1: Base (Build)

- **Base Image Selection**: Uses Node.js version 18.13.0 as the base image and sets the working directory to `/app`.

- **Dependency Installation**: Copies `package.json` and `package-lock.json`, installs dependencies, sets up Prisma, copies the codebase, and builds the application.

## Stage 2: Production (Runtime)

- Defines build-time variables for the user and group, includes a temporary workaround for running Prisma, and sets up the production environment.

- Copies artifacts from the build stage, changes ownership, installs production dependencies, and sets the user to a non-privileged user.

- Exposes port 3000 and specifies the command to start the application.

This Dockerfile simplifies the deployment of your Node.js application and ensures a clean separation between development and production environments.

