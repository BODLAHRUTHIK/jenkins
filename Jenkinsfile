stage('Configure Kubernetes') {
            steps {
                echo "Configuring Kubernetes context for EKS cluster ${EKS_CLUSTER_NAME}"
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                        // Unset potentially conflicting environment variables
                        sh 'unset AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY AWS_SESSION_TOKEN'

                        // Assume the IAM role
                        def assumeRole = sh(script: """
                            aws sts assume-role --role-arn ${ROLE_ARN} --role-session-name ${SESSION_NAME}
                        """, returnStdout: true).trim()
                        
                        def json = readJSON text: assumeRole
                        env.AWS_ACCESS_KEY_ID = json.Credentials.AccessKeyId
                        env.AWS_SECRET_ACCESS_KEY = json.Credentials.SecretAccessKey
                        env.AWS_SESSION_TOKEN = json.Credentials.SessionToken

                        echo "Verifying AWS CLI installation..."
                        sh 'aws --version'
                        
                        // Adding environment variables to debug
                        sh 'env | grep AWS'

                        retry(2) {
                            echo "Fetching AWS caller identity..."
                            sh 'aws sts get-caller-identity'
                        }
                        
                        echo "Updating kubeconfig for the EKS cluster..."
                        sh "aws eks update-kubeconfig --name ${EKS_CLUSTER_NAME} --region ${AWS_REGION}"
                    }
                }
            }
        }
