#!/usr/bin/env groovy

timestamps {
  def repo_slug = 'CancelablePromise'
  def git_commit
  def publish = false

  node('master') {
    fileLoader.withGit('git@github.com:alkemics/lib-groovy-jenkins.git', 'master', 'github-read', '') {
      workflow = fileLoader.load('Workflow')
    }
  }

  node(repo_slug) {
    try {
      def run_or_skip = { condition, closure ->
        if (condition) {
          closure()
        } else {
          echo('skip')
        }
      }

      dir("/home/deploy/${repo_slug}") {
        stage('Initialization') {
          git_commit = common.initialize(repo_slug)
          publish = common.is_master_branch()
        }

        stage('Install') {
          run_or_skip(publish, {
            nodejs.install()
          })
        }

        stage('Publish') {
          run_or_skip(publish, {
            def diff_files = sh(
              script: 'git --no-pager diff origin/master --name-only --diff-filter=d',
              returnStdout: true
            ).split('\n')
            if (diff_files.contains('CHANGELOG.md')) {
              def version = sh(
                script: 'node scripts.js log_publish_version',
                returnStdout: true
              ).trim()
              echo("Publish version ${version}")
              nodejs.publish_release(repo_slug, version)
              common.add_text_badge('published', 'success')
            }
          })
        }

        stage('Finalisation') {
          common.notify_github_pr(repo_slug, git_commit, common.state_success)
        }
      }
    } catch (e) {
      println(e)
      common.notify_github_pr(repo_slug, git_commit, common.state_failed)
      error('Build failed')
    }
  }
}