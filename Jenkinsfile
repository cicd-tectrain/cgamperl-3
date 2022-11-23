// Basis-Pipeline-Schema anlegen

pipeline {

    agent any

    // Define global Environment
    environment {
        INTEGRATION_BRANCH = 'integration'
        PRODUCTION_BRANCH = 'master'
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
                sh 'gradle clean build -x test'
            }

            post {
                success {
                    // Stash build directory for later use
                    stash includes: 'build/', name: 'build'
                }
            }
        }

        stage('Test Integration branch') {

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
                echo 'Testing integration...'
                sh 'gradle test'
            }

            post {
                always {
                    // Collect JUnit test results
                    junit 'build/test-results/**/*.xml'
                }
            }

        }

        stage('Publish artifacts') {
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
                echo 'Publishing artifacts...'
                sh 'ls -la'

                // Upload .jar file to Nexus Maven repository
                nexusArtifactUploader artifacts: [[
                    artifactId: 'at.tectrain.cicd',
                    classifier: '',
                    file: 'build/libs/demo-0.0.1-SNAPSHOT.jar',
                    type: 'jar'
                ]],
                credentialsId: 'nexus_credentials',
                groupId: '',
                nexusUrl: 'nexus:8081/repository/maven-snapshots',
                nexusVersion: 'nexus3',
                protocol: 'http',
                repository: '',
                version: '0.0.1-SNAPSHOT'

            }
        }

        stage('Deploy Integration branch') {

            when {
                branch 'integration'
                beforeAgent true
            }

            environment {
                NEXUS = credentials('nexus_credentials')
            }

            steps {
                echo 'Deploying integration...'

                // Unstash build directory
                unstash 'build'
                sh 'ls -la build'

                // Display info about Docker
                sh 'docker info'
                sh 'docker compose version'
                sh 'docker compose config'

                // Build testing image using docker compose
                sh 'docker compose build testing'

                // Login at Nexus Docker registry
                sh 'echo $NEXUS_PSW | docker login --username $NEXUS_USR --password-stdin nexus:5000'

                // Push image to registry
                sh 'docker compose push testing'

                // Redeploy testing container
                sh 'docker compose up -d --force-recreate testing'
            }

            // Post: Logout Docker
            post {
                always {
                    sh 'docker logout nexus:5000'
                    // Delete /var/jenkins_home/.docker/config.json
                }
            }

        }

        stage('Merge integration into master') {
            steps {
                echo 'Merge into master'
                sh 'ls -la'
                sh 'git branch -a'
                sh 'git checkout ${BRANCH_NAME}'
                sh 'git pull'
                sh 'git checkout ${PRODUCTION_BRANCH}'
                sh 'git merge ${BRANCH_NAME}'
                // Push requires credentials
                withCredentials([
                    gitUsernamePassword(credentialsId: 'github_cicd_pat', gitToolName: 'Default')
                ]) {
                    sh 'git push origin ${PRODUCTION_BRANCH}'
                }

            }
        }


        // ======= Production Stages =======


        stage('Build Production') {

            when {
                branch 'master'
                beforeAgent true
            }

            agent {
                docker {
                    image 'gradle:7.5.1-jdk17-focal'
                }
            }

            steps {
                echo 'Building production...'
                sh 'gradle clean build -x test'
            }

            post {
                success {
                    // Stash build directory for later use
                    stash includes: 'build/', name: 'build-prod'
                }
            }
        }

        stage('Test production') {

            when {
                branch 'master'
                beforeAgent true
            }

            agent {
                docker {
                    image 'gradle:7.5.1-jdk17-focal'
                }
            }

            steps {
                echo 'Testing production...'
                sh 'gradle test'
            }

            post {
                always {
                    junit 'build/test-results/**/*.xml'
                }
            }

        }

        stage('Archive production artifacts') {

            when {
                branch 'master'
                beforeAgent true
            }

            agent {
                docker {
                    image 'gradle:7.5.1-jdk17-focal'
                }
            }

            steps {
                echo 'Publishing production artifacts...'
                sh 'ls -la'

                // Upload .jar file to Nexus Maven repository
                nexusArtifactUploader artifacts: [[
                    artifactId: 'at.tectrain.cicd',
                    classifier: '',
                    file: 'build/libs/demo-0.0.1-SNAPSHOT.jar',
                    type: 'jar'
                ]],
                credentialsId: 'nexus_credentials',
                groupId: '',
                nexusUrl: 'nexus:8081/repository/maven-snapshots',
                nexusVersion: 'nexus3',
                protocol: 'http',
                repository: '',
                version: '0.0.1-SNAPSHOT'

            }

        }

        stage('Deploy Production branch') {

            when {
                branch 'master'
                beforeAgent true
            }

            environment {
                NEXUS = credentials('nexus_credentials')
            }

            steps {
                echo 'Deploying production...'

                // Unstash build directory
                unstash 'build-prod'
                sh 'ls -la build'

                // Display info about Docker
                sh 'docker info'
                sh 'docker compose version'
                sh 'docker compose config'

                // Build testing image using docker compose
                sh 'docker compose build production'

                // Login at Nexus Docker registry
                sh 'echo $NEXUS_PSW | docker login --username $NEXUS_USR --password-stdin nexus:5000'

                // Push image to registry
                sh 'docker compose push production'

                // Redeploy testing container
                sh 'docker compose up -d --force-recreate production'
            }

            // Post: Logout Docker
            post {
                always {
                    sh 'docker logout nexus:5000'
                    // Delete /var/jenkins_home/.docker/config.json
                }
            }
        }

    }


}