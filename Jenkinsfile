pipeline {
    agent any

    environment {
        REMOTE_IP = "54.164.143.230"       // Your EC2 public IP
        REMOTE_USER = "newuser"            // EC2 user
        REMOTE_DIR = "/var/www/html"       // Nginx web root
    }

    stages {

        stage('Clone GitHub Repository') {
            steps {
                echo "Cloning repo..."
                git branch: 'main', url: 'https://github.com/yaswanthkumarch/samplehtml.git', credentialsId: 'github-token'
            }
        }

        stage('Workspace Debug') {
            steps {
                echo "Files in workspace:"
                sh 'ls -la'
            }
        }

        stage('Copy Files to Remote VM') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'remote-vm-pass', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    echo "Copying files to remote VM..."
                    sshpass -p "$PASS" scp -o StrictHostKeyChecking=no -r * $USER@${REMOTE_IP}:/home/$USER/
                    '''
                }
            }
        }

        stage('Deploy to Nginx') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'remote-vm-pass', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    sshpass -p "$PASS" ssh -tt -o StrictHostKeyChecking=no $USER@${REMOTE_IP} "
                        echo 'Cleaning old files...'
                        sudo rm -rf ${REMOTE_DIR}/*
                        echo 'Copying new files...'
                        sudo cp -r /home/$USER/* ${REMOTE_DIR}/
                        echo 'Restarting Nginx...'
                        sudo systemctl restart nginx
                    "
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                echo "Deployment complete! Open http://${REMOTE_IP} to see your site."
            }
        }
    }

    post {
        always { echo "Pipeline finished." }
        success { echo "Website deployed successfully!" }
        failure { echo "Pipeline failed. Check logs." }
    }
}
