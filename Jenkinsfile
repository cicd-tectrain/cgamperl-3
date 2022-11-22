// Basis-Pipeline-Schema anlegen

pipeline {

    agent any

    // Define global Environment
    environment {
        INTEGRATION_BRANCH = 'integration'
    }

    stages {

        stage('Log Environment') {
            steps {
                echo "Local branch: ${BRANCH_NAME}"
                echo "Integration branch: ${INTEGRATION_BRANCH}"
            }
        }

        stage('Build Feature') {
            // Run only on feature branches
            when {
                branch 'feature/*'
                beforeAgent true
            }

            // Diese Stage in einem Docker-Container ausführen
            agent {
                docker {
                    image 'gradle:7.5.1-jdk17-focal'
                }
            }

            steps {
                echo 'Building feature'
                // Cleanup and build proejct (skip tests)
                sh 'gradle clean build -x test'
            }
        }

        stage('Testing Feature') {

            // Run only on feature branches
            when {
                branch 'feature/*'
                beforeAgent true
            }

            // Diese Stage in einem Docker-Container ausführen
            agent {
                docker {
                    image 'gradle:7.5.1-jdk17-focal'
                }
            }

            steps {
                echo 'Testing feature'
                // Run test suite
                sh 'gradle test'
                // List JUnit test files
                sh 'ls -la build/test-results/test'
                // List HTML Test-Report
                sh 'ls -la build/reports/tests/test'
            }

            // Post-build
            post {
                always {
                    // Collect JUnit test results
                    junit 'build/test-results/**/*.xml'
                }

            }

        }

        stage('Integrating Feature') {

            // Run only on feature branches
            when {
                branch 'feature/*'
                beforeAgent true
            }

            steps {
                echo 'Integrating feature'
                sh 'ls -la'
                sh 'git branch -a'
                sh 'git checkout ${BRANCH_NAME}'
                sh 'git checkout ${INTEGRATION_BRANCH}'
                sh 'git merge ${BRANCH_NAME}'
                // Push requires credentials
                withCredentials([
                    gitUsernamePassword(credentialsId: 'github_cicd_pat', gitToolName: 'Default')
                ]) {
                    sh 'git push origin ${INTEGRATION_BRANCH}'
                }

            }
        }

        // ======= Integration Stages =======
        stage('Build Integration branch') {

            when {
                branch 'integration'
                beforeAgent true
            }

            agent {
                docker {
                    image 'gradle:7.5.1-jdk17-focal'
                }
            }

            steps {
                echo 'Building integration...'
            }
        }

        stage('Test Integration branch') {
            steps {
                echo 'Testing integration...'
            }
        }

        stage('Deploy Integration branch') {
            steps {
                echo 'Deploying integration...'
            }
        }


    }


}