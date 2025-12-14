pipeline {
    agent any

    environment {
        DOTNET_CLI_HOME = "${WORKSPACE}/.dotnet"
        DOTNET_ROOT = "${WORKSPACE}/.dotnet"
        PROJECT_NAME = 'Horizons'
        SOLUTION_FILE = 'Horizons.sln'
        BUILD_CONFIG = 'Release'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Restore Dependencies') {
            when {
                branch pattern: "main|develop|feature/.*", comparator: "REGEXP"
            }
            steps {
                echo 'Restoring NuGet packages...'
                sh "dotnet restore ${SOLUTION_FILE}"
            }
        }

        stage('Build') {
            when {
                branch pattern: "main|develop|feature/.*", comparator: "REGEXP"
            }
            steps {
                echo "Building solution in ${BUILD_CONFIG} configuration..."
                sh "dotnet build ${SOLUTION_FILE} --configuration ${BUILD_CONFIG} --no-restore"
            }
        }

        stage('Run Unit Tests') {
    when {
        branch pattern: "main|develop|feature/.*", comparator: "REGEXP"
    }
    steps {
        echo 'Running unit tests...'
        sh "dotnet test Horizons.Tests.Unit/Horizons.Tests.Unit.csproj --configuration ${BUILD_CONFIG} --no-build --verbosity normal --logger 'junit;LogFileName=results.xml' --results-directory TestResults/Unit"
    }
    post {
        always {
            junit 'TestResults/Unit/*.xml'
        }
    }
}

        stage('Run Integration Tests') {
            when {
                branch pattern: "main|develop|feature/.*", comparator: "REGEXP"
            }
            steps {
                echo 'Running integration tests...'
                sh "dotnet test Horizons.Tests.Integration/Horizons.Tests.Integration.csproj --configuration ${BUILD_CONFIG} --no-build --verbosity normal --logger trx --results-directory TestResults/Integration"
            }
            
        }

        stage('Publish') {
            when {
                branch pattern: "main|develop|feature/.*", comparator: "REGEXP"
            }
            steps {
                echo 'Publishing application...'
                sh "dotnet publish ${PROJECT_NAME}/${PROJECT_NAME}.csproj --configuration ${BUILD_CONFIG} --output ./publish --no-build"
            }
        }

        stage('Archive Artifacts') {
            when {
                branch pattern: "main|develop|feature/.*", comparator: "REGEXP"
            }
            steps {
                echo 'Archiving build artifacts...'
                archiveArtifacts artifacts: 'publish/**/*', fingerprint: true
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                echo 'Deploying to production...'
                // Add your deployment steps here
                // Example: sh './deploy-script.sh'
                // Or use specific plugins for Docker, Kubernetes, Azure, AWS, etc.
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
           
        }
        failure {
            echo 'Pipeline failed.'
          
        }
        always {
            echo 'Cleaning up workspace...'
            cleanWs(deleteDirs: true, patterns: [[pattern: '**/TestResults/**', type: 'INCLUDE']])
        }
    }
}