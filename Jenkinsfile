@Library('github.com/fabric8io/fabric8-pipeline-library@master')
def dummy
deployOpenShiftNode(openshiftConfigSecretName: 'fabric8-intcluster-config'){
    properties(
        [
            pipelineTriggers([cron('0 */3 * * *')]),
        ]
    )
    
    def repoNames = 'fabric8io/fabric8-ui,fabric8io/fabric8-planner,fabric8-ui/fabric8-runtime-console'

    def utils = new io.fabric8.Utils()

    // get the repos we want to update
    def ghProjects = splitRepoNames(repoNames)

    def openProjects
    container('clients'){
        def projectList = sh(script: 'oc get projects | grep Active | grep pr-', returnStdout: true).toString().trim()
        openProjects = splitProjectNames(projectList)
    }

    for (ghProject in ghProjects) {
        ghProject = ghProject.toString().trim()

        def openPRs = utils.getOpenPRs(ghProject)
        openPRs = covertToProjectNames(openPRs)

        if (openPRs){
            openProjects.removeAll(openPRs as Object[])
        }

        openPRs = null
    }
    def command = 'oc delete project '
    for (p in openProjects) {
        command = command + ' ' + p

    }
    echo "running: ${command}"
    container('clients'){
        sh command
    }

}

@NonCPS
def splitRepoNames(repoNames) {
    def repos = repoNames.split(',')
    def list = []
    for (name in repos) {
        echo "project to process ${name}"
        list << name
    }
    repos = null
    return list
}

@NonCPS
def splitProjectNames(projectsNames) {
    def projects = projectsNames.split('\\n')
    def list = []
    for (name in projects) {
        list << name.replace("Active", "").trim()
    }
    projects = null
    return list
}

@NonCPS
def covertToProjectNames(openPRs) {
    def list = []
    for (pr in openPRs) {
        list << 'fabric8-ui-pr-' + pr
    }
    return list
}
