
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Automatic trigger test'
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

        stage('Deployment Pre-check') {
            steps {
                sh '''
                    ssh -i /var/lib/jenkins/.ssh/wordpress_deploy \
                    -o StrictHostKeyChecking=accept-new \
                    ec2-user@10.0.2.197 \
                    'test -f /usr/share/nginx/html/wp-config.php &&
                     test -f /usr/share/nginx/html/index.php &&
                     test -d /usr/share/nginx/html/wp-content &&
                     echo "Deployment pre-check passed"'
                '''
            }
        }

        stage('Deploy Test') {
            steps {
                sh '''
                    ssh -i /var/lib/jenkins/.ssh/wordpress_deploy \
                    -o StrictHostKeyChecking=accept-new \
                    ec2-user@10.0.2.197 \
                    'sudo touch /tmp/jenkins-deploy-test &&
                     echo "Deployment test successful"'
                '''
            }
        }

        stage('WordPress Health Check') {
            steps {
                sh '''
                    STATUS=$(ssh -i /var/lib/jenkins/.ssh/wordpress_deploy \
                    -o StrictHostKeyChecking=accept-new \
                    ec2-user@10.0.2.197 \
                    'curl -s -o /dev/null -w "%{http_code}" http://localhost/')

                    echo "WordPress HTTP status: $STATUS"
                    test "$STATUS" = "200"
                '''
            }
        }
    }
}
