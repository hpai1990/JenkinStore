properties = null


//load pipeline properties
def loadProperties(jobname) {
    
    checkout scm
    properties = new Properties()
    File propertiesFile = new File("${workspace}/${jobname}.properties")
    properties.load(propertiesFile.newDataInputStream())
           
}

def removeContainer(imagename) {
    sh """#!/bin/bash
      echo \"Check for existing containers with name ${imagename}\"  
      container_id=`docker ps | grep \"${imagename}\" | awk '{ print \$1 }'`
        
      if [ \"\$container_id\" != \"\" ] ; then
         echo \"Existing container id is \${container_id}\"   
         docker rm -f \$container_id
         echo \"Container has been removed\"
      fi
      """
}
node{
    stage 'Set Up'
    
        loadProperties("${JOB_NAME}")
        //sh 'docker ps | grep "tfangularapp" | awk \'{ print $1 }\' > commandResult'
        removeContainer("${properties.docker_image_name}")
    
      
}

node("${properties.slavenode}"){
    stage 'Checkout'
 	    checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '1ab09c9e-36aa-4285-b73c-7e4d36675372', url: "${properties.scm_url}"]]])
    
    stage 'Build'  
        sh '''
            if [ "${JOB_NAME}" = "uipipeline" ] ; then
                zip -r app.zip .
            elif [ "${JOB_NAME}" = "apppipeline" ] ; then
                mvn install
            else
                echo "No Build tasks"   
             fi
          '''
    stage 'Docker Build and Publish'
        step([$class: 'DockerBuilderPublisher', cleanImages: false, cleanupWithJenkinsJobDelete: false, cloud: 'Docker Colony 2', dockerFileDirectory: '', pullCredentialsId: '', pushCredentialsId: '', pushOnSuccess: true, tagsString: "${properties.docker_repo}/${properties.docker_image_name}:${BUILD_NUMBER}"])
    
    stage 'Deploy Application'
        container_id=step([$class: 'DockerBuilderControl', option: [ $class: 'DockerBuilderControlOptionRun' , cloudName: 'Docker Colony 2' ,image: "${properties.docker_repo}/${properties.docker_image_name}:${BUILD_NUMBER}" ,bindPorts: "${properties.port_bindings}"]])
    
    
}
node{
    stage 'Integration test'
        //checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '1ab09c9e-36aa-4285-b73c-7e4d36675372', url: "https://github.com/NDeeksha/ApiTesting"]]])
        //try{
        //     sh 'mvn install'
        // } 
        //catch (error) {
        //     throw error
        //} 
        //finally {
            //test reporting
        //     junit keepLongStdio: true, testResults: 'target/surefire-reports/junitreports/*.xml'
        //     publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: true, reportDir: 'target/surefire-reports', reportFiles: 'index.html', reportName: 'Integration Test Report', reportTitles: ''])
        build job: 'TestPipeline', parameters: [string(name: 'TestApplication', value: "${properties.test_application}"), string(name: 'TestType', value: "${properties.test_type}"), string(name: 'TestBrowser', value: "${properties.test_browser}"), string(name: 'parallel_execution_mode', value: "${properties.parallel_execution_mode}"), string(name: 'test_classes', value: "${properties.test_classes}")]
        //copyArtifacts filter: 'target/allure-report/*,target/surefire-reports/junitreports/*.xml', fingerprintArtifacts: true, projectName: 'TestPipeline', selector: upstream(allowUpstreamDependencies: false, fallbackToLastSuccessful: false, upstreamFilterStrategy: 'UseGlobalSetting'), target: 'child-artifacts/'    
        copyArtifacts filter: 'htmlreports/*,junitResult.xml', fingerprintArtifacts: true, projectName: 'TestPipeline', selector: lastSuccessful()    
        junit keepLongStdio: true, testResults: '/junitResult.xml'
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: true, reportDir: 'htmlreports/Integration_Test_Report', reportFiles: 'index.html', reportName: 'Integration Test Report', reportTitles: ''])
    //}    
   
        
}

           

