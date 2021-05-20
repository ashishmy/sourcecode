stage 'Checkout'

node {
    checkout(scm)
    stash 'source'
}

stage 'Tests'

parallel(
    knapsack(2) {
        node {
            withCleanup {
                unstash 'source'
                withRvm('ruby-2.5.1') {
                    sh 'rvm use 2.5.1'
                    sh 'bundle install'

                }
            }
        }
    }
)


def withRvm(version, cl) {
    withRvm(version, "executor-${env.EXECUTOR_NUMBER}") {
        cl()
    }
}

def withRvm(version, gemset, cl) {
    RVM_HOME='$HOME/.rvm'
    paths = [

        "$RVM_HOME/rubies/$version/bin",
        "$RVM_HOME/bin",
        "${env.PATH}"
    ]
    def path = paths.join(':')
    withEnv(["PATH=${env.PATH}:$RVM_HOME", "RVM_HOME=$RVM_HOME"]) {
        sh "set +x; source $RVM_HOME/scripts/rvm; rvm use --create --install --binary $version@$gemset"
    }
    withEnv([
        "PATH=$path",
        "GEM_HOME=$RVM_HOME/gems/$version@$gemset",
        "GEM_PATH=$RVM_HOME/gems/$version@$gemset:$RVM_HOME/gems/$version@global",
        "MY_RUBY_HOME=$RVM_HOME/rubies/$version",
        "IRBRC=$RVM_HOME/rubies/$version/.irbrc",
        "RUBY_VERSION=$version"
        ]) {
            sh 'gem install bundler'
            cl()
        }
    }
