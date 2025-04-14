pipeline {
    agent any

    parameters {
        string(name: 'GIT_BRANCH', defaultValue: 'main', description: 'Git branch to use')
        choice(name: 'TARGET_ENV', choices: ['dev', 'test', 'prod'], description: 'Target APIM Environment')
    }

    environment {
        GIT_REPO = 'https://github.com/ericmuchenah/wso2-import-swagger.git'
        SWAGGER_PATH = 'swagger-definitions'
    }

    stages {
        stage('Checkout Swagger Specification') {
            steps {
                git branch: "${params.GIT_BRANCH}",
                    url: "${env.GIT_REPO}"
            }
        }

        
        stage('Login to APIM') {
            steps {
                script {
                    def credsMap = [
                        dev : 'WSO2_CREDENTIALS',
                        test: 'WSO2_CREDENTIALS',
                        prod: 'WSO2_CREDENTIALS'
                    ]

                    def selectedCredsId = credsMap[params.TARGET_ENV]

                    def envMap = [
                        dev : [apim: 'https://localhost:9443', publisher: 'https://localhost:9443/publisher', admin: 'https://localhost:9443/admin'],
                        test: [apim: 'https://localhost:9443', publisher: 'https://localhost:9443/publisher', admin: 'https://localhost:9443/admin'],
                        prod: [apim: 'https://localhost:9443', publisher: 'https://localhost:9443/publisher', admin: 'https://localhost:9443/admin']
                    ]
                    def env = envMap[params.TARGET_ENV]

                    withCredentials([usernamePassword(credentialsId: selectedCredsId, usernameVariable: 'WSO2_USERNAME', passwordVariable: 'WSO2_PASSWORD')]) {
                        sh """
                        apictl remove-env ${params.TARGET_ENV} || true
                        apictl add-env -e ${params.TARGET_ENV} --apim ${env.apim} --admin ${env.admin}
                        apictl login ${params.TARGET_ENV} -u $WSO2_USERNAME -p $WSO2_PASSWORD --insecure --verbose
                        """
                    }
                }
            }
        }

        stage('Import All APIs') {
            steps {
                sh """
                mkdir -p apis-temp
                rm -rf apis-temp/*

                for file in ${env.SWAGGER_PATH}/*.json; do
                  echo "Processing \$file"
                  filename=\$(basename "\$file" .json)
                  apictl init apis-temp/\$filename --oas \$file --verbose
                  apictl import-api -f apis-temp/\$filename -e ${params.TARGET_ENV} --update --verbose
                done
                """
            }
        }
    }

    post {
        success {
            echo 'APIs imported successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check logs.'
        }
    }
}
