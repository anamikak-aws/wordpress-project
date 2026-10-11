
pipeline {
    agent any

    stages {

        stage('Checkout Verification') {
            steps {
                echo 'Checking GitHub source code'
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
                     echo "SSH deployment test successful"'
                '''
            }
        }

        stage('WordPress Health Check - All Instances') {
            steps {
                sh '''
                    for IP in 10.0.2.197 10.0.2.243 10.0.2.146; do

                        echo "Checking WordPress instance: $IP"

                        STATUS=$(ssh -i /var/lib/jenkins/.ssh/key1.pem \
                            -o StrictHostKeyChecking=accept-new \
                            ec2-user@$IP \
                            'curl -s -o /dev/null -w "%{http_code}" --max-time 10 http://localhost/')

                        echo "$IP HTTP Status: $STATUS"

                        test "$STATUS" = "200"
                    done

                    echo "All WordPress instances passed health checks"
                '''
            }
        }

        stage('ALB Health Check') {
            steps {
                sh '''
                    STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
                    --max-time 15 \
                    http://wordpress-alb-1638523939.ap-south-1.elb.amazonaws.com/)

                    echo "ALB HTTP Status: $STATUS"

                    test "$STATUS" = "200"
                    echo "ALB health check passed"
                '''
            }
        }

        stage('Deploy Test File') {
            steps {
                sh '''
                    test -f jenkins-deploy-test.txt

                    ssh -i /var/lib/jenkins/.ssh/wordpress_deploy \
                    -o StrictHostKeyChecking=accept-new \
                    ec2-user@10.0.2.197 \
                    'sudo tee /usr/share/nginx/html/jenkins-deploy-test.txt >/dev/null' \
                    < jenkins-deploy-test.txt

                    ssh -i /var/lib/jenkins/.ssh/wordpress_deploy \
                    -o StrictHostKeyChecking=accept-new \
                    ec2-user@10.0.2.197 \
                    'test "$(curl -s http://localhost/jenkins-deploy-test.txt)" = "Jenkins automatic deployment successful"'

                    echo "Test file deployed and verified successfully"
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: WordPress CI/CD pipeline completed all checks.'
        }

        failure {
            echo 'FAILED: Check the failed stage in the Jenkins console output.'
        }
    }
}
