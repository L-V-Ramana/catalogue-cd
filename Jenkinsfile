pipeline{

    agent{ label 'agent-1'}

    environment{
        appVersion=''
        project='roboshop'
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
                    sed -i "e/image_version/${appVersion}/g" values.yaml
                    helm upgrade --install catalogue -f values.yaml .
                    
                """
                }
            
                
            }
        }
    }
}