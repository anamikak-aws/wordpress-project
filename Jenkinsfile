
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'echo 'Automatic polling test''
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
    }
}
