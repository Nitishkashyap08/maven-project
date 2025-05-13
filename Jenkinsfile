pipeline {
    agent {
        label 'dev-server'
    }

    tools {
        maven 'mymaven'
    }

    parameters {
        choice choices: ['dev', 'prod'], name: 'tests'
    }

    stages {
        stage('build') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }

        stage('test') {
            parallel {
                stage('test A') {
                    agent { label 'dev-server' }
                    steps {
                        echo 'this is test A'
                        sh 'mvn test'
                    }
                }
                stage('test B') {
                    agent { label 'dev-server' }
                    steps {
                        echo 'this is test B'
                        sh 'mvn test'
                    }
                }
            }
        }

        stage('deploy_dev') {
            when {
                expression { params.tests == 'dev' }
                beforeAgent true
            }
            agent { label 'dev-server' }
            steps {
                dir("/var/www/html") {
                    unstash 'mvn-build'
                    sh """
                        cd /var/www/html
                        jar -xvf webapp.war
                    """
                }
            }
        }
    }

    post {
        success {
            dir("webapp/target/") {
                stash name: 'mvn-build', includes: '*.war'
            }
        }
    }
}
