node{
    def buildNumber = BUILD_NUMBER
    
    stage("Git clone"){
        
        git url: 'https://github.com/MithunTechnologiesDevOps/java-web-app-docker.git'
    
    }
    stage("maven"){
        def mavenHome= tool name: 'M2_HOME', type: 'maven'
        sh "${mavenHome}/bin/mvn clean package"
    }
    stage("deploy on nexus"){
        nexusArtifactUploader artifacts: [
            [
                artifactId: 'simple-app', 
                classifier: '', 
                file: 'target/simple-app-1.0.0.war', 
                type: 'war'
                ]
            ], 
            credentialsId: 'nexus3', 
            groupId: 'in.javahome', 
            nexusUrl: '172.31.7.105:8081', 
            nexusVersion: 'nexus3', 
            protocol: 'http', 
            repository: 'simpleapp-release', 
            version: '1.0.0'
        
    }
    stage("Build docker image"){
        sh "docker build -t ram204/java-web-app-docker:${buildNumber} ."
                
    }
    stage("docker login and push"){
        
        withCredentials([string(credentialsId: 'docker-hub-pwd', variable: 'dockerHubPwd')]) {
                  sh "docker login -u ram204 -p ${dockerhubpwd}"
                }
            sh "docker push ram204/java-web-app-docker:${buildNumber}"
        
    }
    stage('run container') {
        
             sshagent(['deploy_docker']) {
                 sh "ssh -o StrictHostKeyChecking=no ec2-user@172.31.12.178 docker rm -f javawebappcontainer || true"
                 sh "ssh -o StrictHostKeyChecking=no ec2-user@172.31.12.178 docker run -d --name javawebappcontainer -p 8080:8080 ram204/java-web-app-docker:${buildNumber}"
               }
      /**stage("Deploy To Kuberates Cluster"){
       kubernetesDeploy(
         configs: 'springBootMongo-PrivateRepo.yml', 
         kubeconfigId: 'KUBERNETES_CLUSTER_CONFIG',
         enableConfigSubstitution: true
        )
     }
	**/
	
    stage("Deploy To Kuberates Cluster"){
        sh 'kubectl create -f springBootMongo-PrivateRepo.yml'
      } 
    
    }
}
