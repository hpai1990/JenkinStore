properties = null

//load pipeline properties
def loadProperties(jobname) {
    
        checkout scm
        properties = new Properties()
        File propertiesFile = new File("${workspace}/${jobname}.properties")
        properties.load(propertiesFile.newDataInputStream())
        //echo "Immediate one ${properties.test}"
    
}
node{
    stage 'Set Up' 
    loadProperties(${JOB_NAME})
      
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
      step([$class: 'DockerBuilderControl', option: [ $class: 'DockerBuilderControlOptionRun' , cloudName: 'Docker Colony 2' ,image: "${properties.docker_repo}/${properties.docker_image_name}:${BUILD_NUMBER}" ,bindPorts: '9000:4200']])
  
}
               

