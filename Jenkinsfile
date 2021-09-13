node('haimaxy-jnlp') {
    stage('Prepare') {
             
           echo "1.Prepare Stage"
           checkout scm
           script {
            build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            if (env.BRANCH_NAME != 'master') {
                build_tag = "${env.BRANCH_NAME}-${build_tag}"
            }
        }
    }
    stage('Test') {
      echo "2.Test Stage"
    }
    stage('Build') {
        echo "3.Build Docker Image Stage"
        sh "docker build -t cnych/jenkins-demo:${build_tag} ."
    }
    stage('Push') {
        echo "4.Push Docker Image Stage"
        withCredentials([usernamePassword(credentialsId: 'harbor', passwordVariable: 'harborPassword', usernameVariable: 'harborUser')]) {
            sh "docker login  harbor.cao.com -u ${harborUser} -p ${harborPassword}"
            sh "docker tag  cnych/jenkins-demo:${build_tag} harbor.cao.com/goharbor/cnych/jenkins-demo:${build_tag}"
            sh "docker push harbor.cao.com/goharbor/cnych/jenkins-demo:${build_tag}"
        }
    }
    stage('Deploy') {
        echo "5. Deploy Stage"
        def userInput = input(
            id: 'userInput',
            message: 'Choose a deploy environment',
            parameters: [
                [
                    $class: 'ChoiceParameterDefinition',
                    choices: "Dev\nQA\nProd",
                    name: 'Env'
                ]
            ]
        )
   
        if (env.BRANCH_NAME == 'master') {
            input "确认要部署线上环境吗？"
        }
        echo "This is a deploy step to ${userInput}"

        sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s.yaml"
        sh "sed -i 's/<BRANCH_NAME>/${env.BRANCH_NAME}/' k8s.yaml"
       
        sh "cat k8s.yaml"
        
        sh "kubectl apply -f k8s.yaml "
    }
}
