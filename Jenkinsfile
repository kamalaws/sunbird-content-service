#!groovy

node('build-slave') {

   currentBuild.result = "SUCCESS"

   try {
           String ANSI_GREEN = "\u001B[32m"
           String ANSI_NORMAL = "\u001B[0m"
           String ANSI_BOLD = "\u001B[1m"
           String ANSI_RED = "\u001B[31m"

         if (params.size() == 0){
            properties([[$class: 'RebuildSettings', autoRebuild: false, rebuildDisabled: false], parameters([string(defaultValue: '', description: '<font color=teal size=2>If you want to build from a tag, specify the tag name. If this parameter is blank, latest commit hash will be used to build</font>', name: 'tag', trim: false)])])

            ansiColor('xterm') {
                println (ANSI_BOLD + ANSI_GREEN + '''\
                        First run of the job. Parameters created. Stopping the current build.
                        Please trigger new build and provide parameters if required.
                        '''.stripIndent().replace("\n"," ") + ANSI_NORMAL)
            }
            return
      }

      stage('Checkout'){
         ansiColor('xterm'){
           if(!env.hub_org){
             println (ANSI_BOLD + ANSI_RED + "Uh Oh! Please set a Jenkins environment variable named hub_org with value as registery/sunbidrded" + ANSI_NORMAL)
             error 'Please resolve the errors and rerun..'
           }
           else
             println (ANSI_BOLD + ANSI_GREEN + "Found environment variable named hub_org with value as: " + hub_org + ANSI_NORMAL)
         }
         cleanWs()
         if(params.tag == ""){
           checkout scm
           commit_hash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
           branch_name = sh(script: 'git name-rev --name-only HEAD | rev | cut -d "/" -f1| rev', returnStdout: true).trim()
         }
         else {
           def scmVars = checkout scm
           checkout scm: [$class: 'GitSCM', branches: [[name: "refs/tags/$params.tag"]],  userRemoteConfigs: [[url: scmVars.GIT_URL]]]
           commit_hash = params.tag
           branch_name = "Blank"
         }
         echo "Git Hash: "+commit_hash
         echo "branch_name: "+branch_name
       }

      stage('Build'){
        env.NODE_ENV = "build"
        print "Environment will be : ${env.NODE_ENV}"
        sh('git submodule update --init')
        sh('git submodule update --init --recursive --remote')
         sh('chmod 777 build.sh')
//         sh("./build.sh ${commit_hash} ${branch_name} ${env.NODE_NAME} ${hub_org}")
         sh("./build.sh ${params.tag} ${branch_name} ${env.NODE_NAME} ${hub_org}")

      }
      stage('ArchiveArtifacts'){
           archiveArtifacts "metadata.json"
      }

    }
    catch (err) {
        currentBuild.result = "FAILURE"
        throw err
    }

}
