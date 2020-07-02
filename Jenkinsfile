import java.text.SimpleDateFormat;

// This pipeline expects a paramater newcolor which will set the newcolor of the newly deployed application.

node {
  // Blue/Green Deployment into Production
  // -------------------------------------
  def project  = ""
  def dest     = "blue"
  def active   = ""
  def newcolor = ""

  switch (BUILD_NUMBER.toInteger() % 10) {
    case 1:
        newcolor="green"
        break
    case 0:
        newcolor="blue"
        break
  }

  stage('Determine Deployment color') {
    // Determine current project
    sh "oc get project|grep -v NAME|awk '{print \$1}' >project.txt"
    project = readFile('project.txt').trim()
    sh "oc get route|grep -v NAME|egrep -v jenkins |awk '{print \$1}' >route.txt"
    route = readFile('route.txt').trim()
    sh "oc get route ${route} -n ${project} -o jsonpath='{ .spec.to.name }' > activesvc.txt"

    // Determine currently active Service
    active = readFile('activesvc.txt').trim()
    if (active == "green") {
      dest = "blue"
    }
    echo "Active svc: " + active
    echo "Dest svc:   " + dest
  }

  stage('Build new version') {
    // Building in this case means simply changing the environment
    // variable newcolor in the deployment configuration.
    // There is not Build Configuration since this is a straight
    // up Docker Image deployment.
    echo "Building ${dest}"
    sh "oc set env dc ${dest} COLOR=${newcolor}"
  }

  stage('Switch over to new Version') {
    input "Switch Production?"
    sh "oc get route|grep -v NAME|egrep -v jenkins |awk '{print \$1}' >route.txt"
    route1 = readFile('route.txt').trim()
    sh 'oc patch route ${route1} -p \'{"spec":{"to":{"name":"' + dest + '"}}}\''
    sh 'oc get route ${route} > oc_out.txt'
    oc_out = readFile('oc_out.txt')
    echo "Current route configuration: " + oc_out
  }
}
