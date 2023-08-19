pipeline{
      // groovvy
      environment{
        // 
        IMAGE_TAG =  sh(returnStdout: true,script: 'echo $image_tag').trim()
        ORIGIN_REPO =  sh(returnStdout: true,script: 'echo $origin_repo').trim()
        REPO =  sh(returnStdout: true,script: 'echo $repo').trim()
        BRANCH =  sh(returnStdout: true,script: 'echo $branch').trim()
      }

      // “slave-pipeline”
      agent{
        node{
          label 'slave-pipeline'
        }
      }

      // "stages" “stage”
      stages{
        // stage， 
        stage('Git'){
          steps{
            git branch: '${BRANCH}', credentialsId: '', url: 'https://github.com/AliyunContainerService/jenkins-demo.git'
          }
        }

        stage('Package'){
          steps{
              container("maven") {
                  sh "mvn package -B -DskipTests"
              }
          }
        }


        // stage,environment groovy环境变量
        stage('Image Build And Publish'){
          steps{
              container("kaniko") {
                  sh "kaniko -f `pwd`/Dockerfile -c `pwd` --destination=${ORIGIN_REPO}/${REPO}:${IMAGE_TAG} --skip-tls-verify"
              }
          }
        }

        // stage, k8s
        stage('Deploy to Kubernetes') {
          steps {
            container('kubectl') {
              step([$class: 'KubernetesDeploy', authMethod: 'certs', apiServerUrl: 'https://kubernetes.default.svc.cluster.local:443', credentialsId:'k8sCertAuth', config: 'deployment.yaml',variableState: 'ORIGIN_REPO,REPO,IMAGE_TAG'])
            }
          }
        }
      }
}
