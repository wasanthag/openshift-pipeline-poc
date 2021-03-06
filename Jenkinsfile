node("maven") 
{
    stage("Checkout") {
     git url: "https://github.com/wasanthag/openshift-pipeline-poc.git", branch: "master"
     }
    stage("Build JAR") {
      sh "mvn clean package"
      stash name:"jar", includes:"target/java-web-server-1.0-SNAPSHOT.jar"
      }
  
    stage("Build Image") {
      unstash name:"jar"
      sh "oc create -f java-web-server-bc.yml"
      sh "oc create is java-web-server -n cicd"  
      sh "oc start-build java-web-server -n cicd"
      timeout(time: 5, unit: 'MINUTES') {
         openshift.withCluster() {
           openshift.withProject() {
             def bc = openshift.selector('bc', "java-web-server")
             echo "Found 1 ${bc.count()} buildconfig"
             def blds = bc.related('builds')
             blds.untilEach {
               echo "Watching new builds created by buildconfig: ${it.names()} : ${it.object().status.phase}"
               return it.object().status.phase == "Complete"
             }
           }
          }  
        }
      }
 }
stage("Deploy") {
                    openshift.withCluster() {
                      openshift.withProject() {
                        def dc = openshift.selector('dc', "java-web-server")
                        dc.rollout().status()
                      }
                    }
                  }
