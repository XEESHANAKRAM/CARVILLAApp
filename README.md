Commit Changes
Update your repository with the new Jenkinsfile (and Dockerfile if not already added):

git add Dockerfile Jenkinsfile
git commit -m "Update Jenkinsfile with Deploy stage for browser access"
git push origin main

4. Jenkins Configuration
Ensure Jenkins is ready to execute the pipeline:

Verify Setup:
Docker: Installed on the Jenkins server, with the Jenkins user in the Docker group (sudo usermod -aG docker jenkins and restart Jenkins).
Plugins: Git Plugin, Docker Pipeline Plugin, and Credentials Plugin installed.
Credentials: Docker Hub credentials stored with ID dockerhub-credentials (Manage Jenkins > Manage Credentials).
Pipeline Job:
Confirm the pipeline job is configured:
Pipeline > Definition: Pipeline script from SCM.
SCM: Git.
Repository URL: https://github.com/XEESHANAKRAM/CARVILLAApp.git.
Branch: main.
Script Path: Jenkinsfile.
Trigger the pipeline (Build Now).

5. Access the Web App in a Browser
After the pipeline runs successfully:

Determine the Server IP:
If running on the Jenkins server, find its IP:
Localhost: localhost or 127.0.0.1 (if accessing from the server itself).
Remote: Use the server’s public or private IP (e.g., 192.168.x.x or a public IP like 34.123.45.67).
If deploying to a separate server, use that server’s IP.
Open the Browser:
Navigate to http://<server-ip>:8000.
Example:
Local: http://localhost:8000
Remote: http://192.168.1.100:8000 or http://34.123.45.67:8000
You should see the content of index.html rendered by Nginx.
Verify Container:
On the server, check the running container:

docker ps
Look for my-html-container with ports 0.0.0.0:8000->80/tcp.
View logs if needed:
docker logs my-html-container

6. Troubleshooting Browser Access
If you can’t access the web app in the browser:

Firewall:
Ensure port 8000 is open on the server:
sudo ufw allow 8000
Or, for cloud servers (e.g., AWS, GCP):
AWS: Add a rule to the EC2 security group for port 8000 (inbound TCP).
GCP: Add a firewall rule for port 8000.
Test connectivity:
curl http://localhost:8000
If this works locally but not remotely, the firewall or security group is likely blocking port 8000.
Port Conflict:
Check if port 8000 is already in use:
sudo netstat -tuln | grep 8000
If occupied, stop the conflicting service or change EXTERNAL_PORT in the Jenkinsfile (e.g., to 8080).
Container Issues:
Verify the container is running:
docker ps -a
If the container exited, check logs:
docker logs my-html-container
Ensure the Dockerfile and index.html are correct.
Network Access:
If using a remote server, confirm it has a public IP or is accessible via your network.
For local testing, use localhost or 127.0.0.1.
Nginx Configuration:
If index.html doesn’t load, verify Nginx is serving it:
docker exec my-html-container cat /usr/share/nginx/html/index.html
Ensure index.html has valid HTML content.

7. Optional: Deploy to a Separate Server
If you want to deploy the container to a separate server (not the Jenkins server):

Update Deploy Stage:

Use SSH to connect to the target server and run the container.
Install the SSH Agent Plugin in Jenkins.
Add SSH credentials in Manage Jenkins > Manage Credentials (e.g., ID: ssh-credentials).
Modified Deploy Stage:
Replace the Deploy stage with:
stage('Deploy') {
    steps {
        script {
            withCredentials([sshUserPrivateKey(credentialsId: 'ssh-credentials', keyFileVariable: 'SSH_KEY', usernameVariable: 'SSH_USER')]) {
                sh '''
                    ssh -i $SSH_KEY $SSH_USER@<target-server-ip> \\
                    "docker stop ${CONTAINER_NAME} || true && \\
                     docker rm ${CONTAINER_NAME} || true && \\
                     docker pull ${DOCKER_IMAGE}:${IMAGE_TAG} && \\
                     docker run --name ${CONTAINER_NAME} -d -p ${EXTERNAL_PORT}:80 ${DOCKER_IMAGE}:${IMAGE_TAG}"
                '''
            }
        }
    }
}
Replace <target-server-ip> with the target server’s IP (e.g., 34.123.45.67).
Ensure the target server has Docker installed and the SSH user has Docker permissions.
Access:

Open http://<target-server-ip>:8000 in the browser.
8. Enhancements
Test Container in Pipeline:
Add a test after deployment to verify the app is accessible:
stage('Verify Deployment') {
    steps {
        sh '''
            sleep 5
            curl -f http://localhost:${EXTERNAL_PORT} || exit 1
        '''
    }
}
Add this stage after Deploy.

Dynamic Ports:
If port 8000 is problematic, make EXTERNAL_PORT configurable via Jenkins parameters.
HTTPS:
For production, configure Nginx with SSL (e.g., using Let’s Encrypt) and update the Dockerfile.
Cleanup:
Remove old images on the server:
docker image prune -f


9. Verify in Browser
After the pipeline completes, open http://<server-ip>:8000 (e.g., http://localhost:8000 or http://34.123.45.67:8000).
You should see index.html rendered (e.g., if it contains <h1>Hello</h1>, that text appears).
If the page doesn’t load, use the troubleshooting steps above.
