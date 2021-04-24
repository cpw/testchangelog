@Library('forge-shared-library')_

pipeline {
    agent {
        docker {
            image 'gradle:jdk8'
        }
    }
    environment {
        GRADLE_ARGS = '--no-daemon'
    }

    stages {
        stage('buildandtest') {
            steps {
                echo gradleVersion()
            }
        }        
    }
}
