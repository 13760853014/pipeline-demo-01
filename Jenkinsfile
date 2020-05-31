// 需要在jenkins的Credentials设置中配置jenkins-harbor-creds、jenkins-k8s-config参数
import hudson.model.*;

pipeline {
    agent any
    environment {
        HARBOR_CREDS = credentials('jenkins-harbor-creds')
        //K8S_CONFIG = credentials('jenkins-k8s-config')
        GIT_TAG = env.BUILD_NUMBER//sh(returnStdout: true,script: 'git describe --tags --always').trim()
        GIT_REPO = sh(label: '', returnStdout: true, script: 'git config remote.origin.url').trim()
    }
    parameters {
        string(name: 'HARBOR_HOST', defaultValue: '47.106.81.219', description: 'harbor仓库地址')
        string(name: 'DOCKER_IMAGE', defaultValue: 'tssp/pipeline-demo', description: 'docker镜像名')
        string(name: 'APP_NAME', defaultValue: 'pipeline-demo', description: 'k8s中标签名')
        //string(name: 'K8S_NAMESPACE', defaultValue: 'demo', description: 'k8s的namespace名称')
    }
    stages {

        stage("Preparation") {
			steps{
				script {
					print "gitlab branch: " + env.gitlabSourceBranch
                    print "gitlab GIT_REPO: " + env.GIT_REPO
                    print "gitlab branchName: " + env.gitlabSourceBranch
                    print "gitlab username: " + env.gitlabUserName
				}

				sh "echo PROJECT = ${params.PROJECT}"
                sh "echo INSTALL = ${params.INSTALL}"
                sh "echo ENV = ${params.ENV}"
                sh "echo FORCE = ${params.FORCE}"
                sh "echo INIT = ${params.INIT}"

                sh "echo WORKSPACE = $WORKSPACE"
                sh "echo BUILD_ID = $BUILD_ID"

                sh 'pwd'

                sh "echo BUILD_NUMBER = $BUILD_NUMBER"
                sh "echo JOB_NAME = $JOB_NAME"
                sh "echo JOB_BASE_NAME = $JOB_BASE_NAME"
                sh "echo BUILD_TAG = $BUILD_TAG"
                sh "echo EXECUTOR_NUMBER = $EXECUTOR_NUMBER"
                sh "echo NODE_NAME = $NODE_NAME"
                sh "echo NODE_LABELS = $NODE_LABELS"
                sh "echo JENKINS_HOME = $JENKINS_HOME"
                sh "echo JENKINS_URL = $JENKINS_URL"
                sh "echo BUILD_URL = $BUILD_URL"
                sh "echo JOB_URL = $JOB_URL"
			}
		}
        stage('Maven Build') {
            when { expression { env.GIT_TAG != null } }
            agent {
                docker {
                    image 'maven:3-jdk-8-alpine'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn clean package -Dfile.encoding=UTF-8 -DskipTests=true'
                stash includes: 'target/*.jar', name: 'app'
            }

        }
        stage('Docker Build') {
            when { 
                allOf {
                    expression { env.GIT_TAG != null }
                }
            }
            agent any
            steps {
                unstash 'app'
                sh "docker login -u ${HARBOR_CREDS_USR} -p ${HARBOR_CREDS_PSW} ${params.HARBOR_HOST}"
                sh "docker build --build-arg JAR_FILE=`ls target/*.jar |cut -d '/' -f2` -t ${params.HARBOR_HOST}/${params.DOCKER_IMAGE}:${GIT_TAG} ."
                sh "docker push ${params.HARBOR_HOST}/${params.DOCKER_IMAGE}:${GIT_TAG}"
                //sh "docker rmi ${params.HARBOR_HOST}/${params.DOCKER_IMAGE}:${GIT_TAG}"
            }
            
        }
        stage('Deploy') {
            when { 
                allOf {
                    expression { env.GIT_TAG != null }
                }
            }
            //agent {
            //    docker {
            //        image 'lwolf/helm-kubectl-docker'
            //    }
            //}
            steps {
                sh "docker run -p 40080:40080 ${params.HARBOR_HOST}/${params.DOCKER_IMAGE}:${GIT_TAG} &"
                //sh "mkdir -p ~/.kube"
                //sh "echo ${K8S_CONFIG} | base64 -d > ~/.kube/config"
                //sh "sed -e 's#{IMAGE_URL}#${params.HARBOR_HOST}/${params.DOCKER_IMAGE}#g;s#{IMAGE_TAG}#${GIT_TAG}#g;s#{APP_NAME}#${params.APP_NAME}#g;s#{SPRING_PROFILE}#k8s-test#g' k8s-deployment.tpl > k8s-deployment.yml"
                //sh "kubectl apply -f k8s-deployment.yml --namespace=${params.K8S_NAMESPACE}"
            }
            
        }
        
    }
}
