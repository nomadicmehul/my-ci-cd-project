### Prerequisites
1. **[Docker Desktop](https://www.docker.com/products/docker-desktop/)**: Installed and running on your local machine.
2. **[GitHub Account](http://github.com/)**: For version control and CI/CD pipeline.
3. **[Docker Hub Account](http://hub.docker.com/)**: For storing Docker images.
4. **Basic Knowledge**: Familiarity with Docker, Git, and CI/CD concepts.

### Step 1: Create a Simple Application
Let's start by creating a simple Node.js application.

1. **Create a new directory** for your project:
   ```bash
   mkdir my-ci-cd-project
   cd my-ci-cd-project
   ```

2. **Initialize a new Node.js project**:
   ```bash
   npm init -y
   ```

3. **Create a simple `index.js` file**:
   ```javascript
   // index.js
   const express = require('express');
   const app = express();
   const port = 3000;

   app.get('/', (req, res) => {
     res.send('Hello, Docker!');
   });

   app.listen(port, () => {
     console.log(`App listening at http://localhost:${port}`);
   });
   ```

4. **Install Express**:
   ```bash
   npm install express
   ```

5. **Create a `Dockerfile`**:
   ```Dockerfile
   # Use an official Node.js runtime as a parent image
   FROM node:14

   # Set the working directory in the container
   WORKDIR /usr/src/app

   # Copy package.json and package-lock.json
   COPY package*.json ./

   # Install dependencies
   RUN npm install

   # Copy the rest of the application code
   COPY . .

   # Expose the port the app runs on
   EXPOSE 3000

   # Define the command to run the app
   CMD ["node", "index.js"]
   ```

6. **Create a `.dockerignore` file**:
   ```
   node_modules
   npm-debug.log
   ```

### Step 2: Build and Test the Docker Image Locally
1. **Build the Docker image**:
   ```bash
   docker build -t my-ci-cd-project .
   ```

2. **Run the Docker container**:
   ```bash
   docker run -p 3000:3000 my-ci-cd-project
   ```

3. **Test the application**:
   Open your browser and navigate to `http://localhost:3000`. You should see "Hello, World!".

### Step 3: Set Up GitHub Repository
1. **Initialize a Git repository**:
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   ```

2. **Create a new repository on GitHub** and push your code:
   ```bash
   git remote add origin https://github.com/your-username/my-ci-cd-project.git
   git branch -M main
   git push -u origin main
   ```

### Step 4: Set Up GitHub Actions for CI/CD
1. **Create a `.github/workflows/ci-cd.yml` file**:
   ```yaml
   name: CI/CD Pipeline

   on:
     push:
       branches:
         - main
     pull_request:
       branches:
         - main

   jobs:
     build:
       runs-on: ubuntu-latest

       steps:
       - name: Checkout code
         uses: actions/checkout@v2

       - name: Login to Docker Hub
         uses: docker/login-action@v1
         with:
           username: ${{ secrets.DOCKER_HUB_USERNAME }}
           password: ${{ secrets.DOCKER_HUB_TOKEN }}

       - name: Build Docker image
         run: docker build -t my-ci-cd-project .

       - name: Tag Docker image
         run: docker tag my-ci-cd-project ${{ secrets.DOCKER_HUB_USERNAME }}/my-ci-cd-project:latest

       - name: Push Docker image
         run: docker push ${{ secrets.DOCKER_HUB_USERNAME }}/my-ci-cd-project:latest

     deploy:
       runs-on: ubuntu-latest
       needs: build

       steps:
       - name: Checkout code
         uses: actions/checkout@v2

       - name: Login to Docker Hub
         uses: docker/login-action@v1
         with:
           username: ${{ secrets.DOCKER_HUB_USERNAME }}
           password: ${{ secrets.DOCKER_HUB_TOKEN }}

       - name: Pull Docker image
         run: docker pull ${{ secrets.DOCKER_HUB_USERNAME }}/my-ci-cd-project:latest

       - name: Run Docker container
         run: docker run -d -p 3000:3000 ${{ secrets.DOCKER_HUB_USERNAME }}/my-ci-cd-project:latest
   ```

2. **Add Docker Hub credentials to GitHub Secrets**:
   - Go to your GitHub repository.
   - Navigate to **Settings** > **Secrets** > **New repository secret**.
   - Add `DOCKER_HUB_USERNAME` and `DOCKER_HUB_TOKEN` as secrets.

### Step 5: Test the CI/CD Pipeline
1. **Push changes to the `main` branch**:
   ```bash
   git add .
   git commit -m "Add CI/CD pipeline"
   git push origin main
   ```

2. **Monitor the GitHub Actions workflow**:
   - Go to the **Actions** tab in your GitHub repository.
   - You should see the workflow running, building the Docker image, and pushing it to Docker Hub.

3. **Verify the deployment**:
   - Once the workflow completes, your Docker image should be deployed and running.
   - You can verify by accessing the application at the appropriate URL.

### Conclusion
You now have a basic CI/CD pipeline set up using Docker Desktop, GitHub Actions, and Docker Hub. This pipeline automates the process of building, testing, and deploying your Dockerized application. You can further enhance this setup by adding more stages, such as running tests, linting, or deploying to different environments.

Feel free to ask if you have any questions or need further assistance!
