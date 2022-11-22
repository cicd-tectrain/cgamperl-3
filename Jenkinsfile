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
                    image '7.5.1-jdk17-focal'
                }
            }

            steps {
                echo 'Building feature'
            }
        }

        stage('Testing Feature') {

            // Run only on feature branches
            when {
                branch 'feature/*'
                beforeAgent true
            }

            steps {
                echo 'Testing feature'
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