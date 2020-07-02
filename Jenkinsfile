import java.text.SimpleDateFormat;

// This pipeline expects a paramater newcolor which will set the newcolor of the newly deployed application.

node {
  // Blue/Green Deployment into Production
  // -------------------------------------
  def project  = ""
  def dest     = ""
  def active   = ""

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
    } else (active == "blue") {
      dest = "green"
    }
    echo "Active svc: " + active
    echo "Dest svc:   " + dest
  }

  stage('Switch over to new Version') {
    input "Switch Production?"
    sh "oc get route|grep -v NAME|egrep -v jenkins |awk '{print \$1}' >route.txt"
    route = readFile('route.txt').trim()
    sh 'oc patch route bluegreen -p \'{"spec":{"to":{"name":"' + dest + '"}}}\''
    sh "oc get route ${route} > oc_out.txt"
    oc_out = readFile('oc_out.txt')
    echo "Current route configuration: " + oc_out
  }
}
