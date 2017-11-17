properties = null


def loadProperties() {
    
        checkout scm
        properties = new Properties()
        File propertiesFile = new File("${workspace}/pipeline.properties")
        properties.load(propertiesFile.newDataInputStream())
        echo "Immediate one ${properties.test}"
    
}
node{
    stage ('prepare') {
            

            steps {
                script {
                    loadProperties()
                    echo "Later one ${properties.test}"
                }
            }
        }
}
