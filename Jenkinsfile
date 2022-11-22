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