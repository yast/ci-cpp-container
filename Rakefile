require "yast/rake"

Yast::Tasks.configuration do |conf|
  # lets ignore license check for now
  conf.skip_license_check << /.*/

  conf.package_name = "ci-cpp-container"
  conf.obs_target = "containers"
end

# do not create a tarball, this project builds a Docker container
Rake::Task["tarball"].clear_actions

# building at Jenkins does not work for some reason, disable it for now
Rake::Task["osc:build"].clear if ENV["JENKINS_HOME"]

