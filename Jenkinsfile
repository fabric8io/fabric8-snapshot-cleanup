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

    // check the repos for open PRs
    def ghProjects = splitRepoNames(repoNames)

    // get a list of openshft project names that are pull requests
    def osProjects
    container('clients'){
        def projectList = sh(script: 'oc get projects | grep Active | grep pr-', returnStdout: true).toString().trim()
        osProjects = splitProjectNames(projectList)
    }

    // remove any names that still have open PRs in github
    for (ghProject in ghProjects) {
        ghProject = ghProject.toString().trim()

        def openPRs = utils.getOpenPRs(ghProject)
        openPRs = covertToProjectNames(openPRs)

        if (openPRs){
            osProjects.removeAll(openPRs as Object[])
        }

        openPRs = null
    }
    if (osProjects){
        def command = 'oc delete project '
        for (p in osProjects) {
            command = command + ' ' + p

        }
        echo "running: ${command}"
        container('clients'){
            sh command
        }
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
