template = '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: packer
  name: packer
spec:
  containers:
  - command:
    - sleep
    - "3600"
    image: hashicorp/packer
    name: packer
    '''

def buildNumber = env.BUILD_NUMBER

properties([
    parameters(
        [choice(choices: ['dev', 'qa', 'stage', 'prod'], name: 'environment')
        ])
        ])

if (params.environment == "dev" ) {
    region = "us-east-1"
}
else if (params.environment == "qa" ) {
    region = "us-east-2"
}

else if (params.environment == "stage" ) {
    region = "us-west-1"
}

else {
    region = "us-west-2"
}



podTemplate(cloud: 'kubernetes', label: 'packer', showRawYaml: false, yaml: template) {
    node("packer"){
        container("packer"){
        withCredentials([usernamePassword(credentialsId: 'aws-creds', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
            withEnv(["AWS_REGION=${region}"]) {

            stage("Git Clone"){
                git branch: 'main', url: 'https://github.com/kaizenacademy/packer1.git'
            }
            
            stage("packer build"){
                sh "packer build -var 'jenkins_build_number=${buildNumber}' packer.pkr.hcl"
            }
        }
        }
        }
    }
}
