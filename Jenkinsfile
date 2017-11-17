properties = null


def loadProperties() {
    
        checkout scm
        properties = new Properties()
        File propertiesFile = new File("${workspace}/pipeline.properties")
        properties.load(propertiesFile.newDataInputStream())
        echo "Immediate one ${properties.test}"
    
}
node{
    stage 'prepare' 
      loadProperties()
      echo "Later one ${properties.test}"
}
node("${properties.slavenode}"){
    stage 'Checkout'
 		checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '1ab09c9e-36aa-4285-b73c-7e4d36675372', url: 'https://github.com/tf-poc/angular/']]])
}
               

