pipeline{

    agent{ label 'agent-1'}

    environment{
        appVersion=''
        project='roboshop'
        component = 'catalogue'
        region='us-east-1'

    }

    parameters{
        string(name: 'appVersion', description: 'imageversion')
        choice(name: 'deploy_to', choices:['dev','qa','prod'],description: 'pick the environment')
    }

    stages{
        stage('deploy'){
            steps{
                withAWS(credentials: 'aws-auth',region: 'us-east-1'){
                        sh"""
                    aws eks update-kubeconfig --region $region --name "$project-${params.deploy_to}"
                    kubectl get pods
                    kubectl apply -f 00-namespace.yaml
                    sed -i "e/image_version/${env.appVersion}/g" values.yaml
                    helm upgrade --install catalogue -f values.yaml .
                    
                """
                }
                         
            }
        }

        stage('roll back'){
            steps{
                script{
                    withAWS(credentials:'aws-auth', region: 'us-east-1')
                     def rolloutStatus = sh(
                            script: """kubectl rollout status deployment/catalogue --timeout=30s -n roboshop || echo failed""",
                            returnStdout: true
                        ).trim()

                         if(rolloutStatus.contains("successfully rolled out")){
                        echo "deployment is successful"
                }
                else{
                    sh """
                        helm rollback $component -n $project
                    """
                    def rollbackstatus = sh( script: """ kubectl rollout status deployment/catalogue --timeout=30s
                     -n roboshop||echo failed """,
                    returnstdout:true).trim()

                    if(rollbackstatus.contains("successfully rolled out")){
                        error "deployment failed, rolleback success"
                    }
                    else{
                        error "deployment failed, rollback failed, application down"
                    }
                }
                }

               
            }
        }
    }

    post{
        always{
            deleteDir()

        }

        success{
            echo "pipeline is successful"
        }

        failure{
            echo "pipeline is failed"
        }
    }
}