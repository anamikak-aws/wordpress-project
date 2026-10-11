
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Automatic polling test'
                sh 'git rev-parse --short HEAD'
            }
        }

        stage('Validate WordPress Files') {
            steps {
                sh '''
                    test -f index.php
                    test -f wp-config-sample.php
                    test ! -f wp-config.php
                    echo "WordPress files validated successfully"
                '''
            }
        }

        stage('Deploy Test') {
            steps {
                sh '''
                    ssh -i /var/lib/jenkins/.ssh/wordpress_deploy \
                    -o StrictHostKeyChecking=accept-new \
                    ec2-user@10.0.2.197 \
                    'sudo touch /tmp/jenkins-deploy-test && echo "Deployment test successful"'
                '''
            }
        }
    }
}

