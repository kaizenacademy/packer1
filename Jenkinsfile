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

if (env.BRANCH == "dev" ) {
    region = "us-east-1"
}
else if (env.BRANCH == "qa" ) {
    region = "us-east-2"
}

else if (env.BRANCH == "master" ) {
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

                build job: 'terraform-ec2', parameters: [string(name: 'action', value: 'apply'), string(name: 'region', value: "${region}"), string(name: 'ami_name', value: "my-amy-${buildNumber}"), string(name: 'az', value: "${region}b")]
            }
        }
        }
        }
    }
}
