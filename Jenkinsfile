pipeline {
    agent any

    parameters {
        string(name: 'GIT_BRANCH', defaultValue: 'main', description: 'Git branch to use')
        choice(name: 'TARGET_ENV', choices: ['dev', 'test', 'prod'], description: 'Target APIM Environment')
    }

    environment {
        GIT_REPO = 'https://github.com/ericmuchenah/wso2-import-swagger.git'
        SWAGGER_PATH = 'api/swagger.json' // relative path inside repo
        API_PROJECT_NAME = 'test-api'
    }

    stages {
        stage('Checkout Swagger Specification') {
            steps {
                git branch: "${params.GIT_BRANCH}",
                    url: "${env.GIT_REPO}"
            }
        }

        stage('Initialize API Project') {
            steps {
                sh """
                rm -rf ${API_PROJECT_NAME} || true
                apictl init ${API_PROJECT_NAME} --oas ${SWAGGER_PATH} --verbose
                """
            }
        }

        stage('Import API to WSO2 APIM') {
            steps {
                sh """
                apictl import-api -f ${API_PROJECT_NAME} -e ${params.TARGET_ENV} --update --verbose
                """
            }
        }
    }

    post {
        success {
            echo 'API imported successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check logs.'
        }
    }
}
