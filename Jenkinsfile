pipeline {
    agent any

    parameters {
        string(name: 'GIT_BRANCH', defaultValue: 'main', description: 'Git branch to use')
        choice(name: 'TARGET_ENV', choices: ['dev', 'test', 'prod'], description: 'Target APIM Environment')
    }

    environment {
        GIT_REPO = 'https://github.com/ericmuchenah/wso2-import-swagger.git'
        SWAGGER_PATH = 'swagger-definitions'
        ENDPOINT_FILE = "endpoints.${params.TARGET_ENV}.json"
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
                        apictl remove env ${params.TARGET_ENV} || true
                        apictl add env ${params.TARGET_ENV} --apim ${env.apim} --admin ${env.admin}
                        apictl login ${params.TARGET_ENV} -u $WSO2_USERNAME -p $WSO2_PASSWORD --insecure --verbose
                        """
                    }
                }
            }
        }

        stage('Import All Swagger APIs') {
            steps {
                sh """
                mkdir -p apis-temp
                rm -rf apis-temp/*

                if [ -d "${env.SWAGGER_PATH}" ]; then
                  for file in ${env.SWAGGER_PATH}/*.json; do
                    if [ -f "\$file" ]; then
                      echo "Processing \$file"
                      filename=\$(basename "\$file" .json)
                      apictl init apis-temp/\$filename --oas "\$file" --verbose
                      
                      prod=\$(grep -A2 "\"$filename\"" endpoints.\${params.TARGET_ENV}.json | grep production | awk -F '"' '{print \$4}')
                      sandbox=\$(grep -A2 "\"$filename\"" endpoints.\${params.TARGET_ENV}.json | grep sandbox | awk -F '"' '{print \$4}')

                      cat > apis-temp/\$name/api_params.yaml <<EOF
                    environments:
                      - name: \${params.TARGET_ENV}
                        endpoints:
                          production:
                            url: "\$prod"
                          sandbox:
                            url: "\$sandbox"
                    EOF

                      apictl import-api -f apis-temp/\$filename -e ${params.TARGET_ENV}  --insecure --update --verbose
                    fi
                  done
                  apictl logout ${params.TARGET_ENV} --insecure
                else
                  echo "Folder '${env.SWAGGER_PATH}' does not exist. Skipping API import."
                  exit 1
                fi

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
