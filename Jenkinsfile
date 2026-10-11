
pipeline {
    agent any

    environment {
        WP_PATH = '/usr/share/nginx/html'
        DEPLOY_KEY = '/var/lib/jenkins/.ssh/key1.pem'
        DEPLOY_USER = 'ec2-user'
        WP_INSTANCES = '10.0.2.197 10.0.2.243 10.0.2.146'
        ALB_URL = 'http://wordpress-alb-1638523939.ap-south-1.elb.amazonaws.com'
    }

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
                    test -f wp-includes/version.php
                    test -d wp-content
                    test ! -f wp-config.php
                    echo "Source files validated"
                '''
            }
        }

        stage('Pre-deployment Checks') {
            steps {
                sh '''
                    for IP in $WP_INSTANCES; do
                        echo "Checking $IP"
                        ssh -i "$DEPLOY_KEY" \
                            -o StrictHostKeyChecking=accept-new \
                            "$DEPLOY_USER@$IP" \
                            "sudo test -f $WP_PATH/wp-config.php &&
                             sudo test -f $WP_PATH/index.php"
                    done
                    echo "All instances have existing WordPress configuration"
                '''
            }
        }

        stage('Deploy Application Files') {
            steps {
                sh '''
                    set -eu

                    # Deploy only application content tracked in Git.
                    # Do not delete server files absent from Git.
                    # Never copy wp-config.php from GitHub.
                    for IP in $WP_INSTANCES; do
                        echo "Deploying to $IP"

                        rsync -av \
                            --exclude='/.git/' \
                            --exclude='/wp-config.php' \
                            --exclude='/wp-content/uploads/' \
                            --exclude='/wp-content/cache/' \
                            --exclude='/wp-content/upgrade/' \
                            -e "ssh -i $DEPLOY_KEY -o StrictHostKeyChecking=accept-new" \
                            ./ "$DEPLOY_USER@$IP:/tmp/wordpress-deploy/"

                        ssh -i "$DEPLOY_KEY" \
                            -o StrictHostKeyChecking=accept-new \
                            "$DEPLOY_USER@$IP" \
                            'sudo rsync -av \
                              --exclude="wp-config.php" \
                              --exclude="wp-content/uploads/" \
                              --exclude="wp-content/cache/" \
                              --exclude="wp-content/upgrade/" \
                              /tmp/wordpress-deploy/ \
                              /usr/share/nginx/html/ &&
                             sudo rm -rf /tmp/wordpress-deploy'
                    done
                '''
            }
        }

        stage('WordPress Health Check') {
            steps {
                sh '''
                    set -eu
                    for IP in $WP_INSTANCES; do
                        STATUS=$(ssh -i "$DEPLOY_KEY" \
                            -o StrictHostKeyChecking=accept-new \
                            "$DEPLOY_USER@$IP" \
                            'curl -s -o /dev/null -w "%{http_code}" \
                             --max-time 10 http://localhost/')
                        echo "$IP HTTP $STATUS"
                        test "$STATUS" = "200"
                    done
                '''
            }
        }

        stage('ALB Health Check') {
            steps {
                sh '''
                    STATUS=$(curl -s -o /dev/null -w "%{http_code}" \
                        --max-time 15 "$ALB_URL/")
                    echo "ALB HTTP $STATUS"
                    test "$STATUS" = "200"
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Deployment and health checks passed.'
        }
        failure {
            echo 'FAILED: Check the failed stage in Jenkins Console Output.'
        }
    }
}
