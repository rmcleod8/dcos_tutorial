def gitCommit() {
    sh "git rev-parse HEAD > GIT_COMMIT"
    def gitCommit = readFile('GIT_COMMIT').trim()
    sh "rm -f GIT_COMMIT"
    return gitCommit
}

node {
    // Checkout source code from Git
    stage 'Checkout'
    checkout scm

    // Build Docker image
    stage 'Build'
    sh "docker build -t lobsters/dcos_tutorial:${gitCommit()} ."

    // Log in and push image to GitLab
    stage 'Publish'
    withCredentials(
        [[
            $class: 'UsernamePasswordMultiBinding',
            credentialsId: 'dockerhub-lobsters',
            passwordVariable: 'DOCKERHUB_PASSWORD',
            usernameVariable: 'DOCKERHUB_USERNAME'
        ]]
    ) {
        sh "docker login -u lobsters -p velocitypassword -e richard.mcleod@gmail.com"
        sh "docker push lobsters/dcos_tutorial:${gitCommit()}"
    }
// Deploy
    stage 'Deploy'

    marathon(
        url: 'http://marathon.mesos:8080',
        forceUpdate: false,
        credentialsId: 'dcos-token',
        filename: 'marathon.json',
        appId: 'nginx-mesosphere-lobsters',
        docker: "lobsters/dcos_tutorial:${gitCommit()}".toString()
    )
}
