properties = null
container_id = null

//load pipeline properties
def loadProperties(jobname) {
    
        checkout scm
        properties = new Properties()
        File propertiesFile = new File("${workspace}/"+jobname+".properties")
        properties.load(propertiesFile.newDataInputStream())
           
}
node{
    stage 'Set Up'
    echo "test step"
    loadProperties("${JOB_NAME}")
    sh 'docker ps | grep "tfangularapp" | awk \'{ print $1 }\' > commandResult'
    container_id = readFile('commandResult').trim()
    echo "Container id is : ${container_id}"
      
}

node("${properties.slavenode}"){
    stage 'Checkout'
 	  checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '1ab09c9e-36aa-4285-b73c-7e4d36675372', url: "${properties.scm_url}"]]])
    
    stage 'Build'  
      sh '''
      if [ "${JOB_NAME}" = "uipipeline" ] ; then
         zip -r app.zip .
      fi
      '''
    stage 'Docker Build and Publish'
      step([$class: 'DockerBuilderPublisher', cleanImages: false, cleanupWithJenkinsJobDelete: false, cloud: 'Docker Colony 2', dockerFileDirectory: '', pullCredentialsId: '', pushCredentialsId: '', pushOnSuccess: true, tagsString: "${properties.docker_repo}/${properties.docker_image_name}:${BUILD_NUMBER}"])
    
    stage 'Deploy Application'
      container_id=step([$class: 'DockerBuilderControl', option: [ $class: 'DockerBuilderControlOptionRun' , cloudName: 'Docker Colony 2' ,image: "${properties.docker_repo}/${properties.docker_image_name}:${BUILD_NUMBER}" ,bindPorts: '9000:4200']])
      
}               

