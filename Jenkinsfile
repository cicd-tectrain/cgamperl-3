// Basis-Pipeline-Schema anlegen

pipeline {

    agent any

    stages {

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
            }
        }

    }


}