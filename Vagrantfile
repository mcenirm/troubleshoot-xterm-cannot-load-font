# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "bento/centos-7"
  config.vm.provision "shell" do |shell|
    shell.path = "experiment_4_xterm_no_error_message_in_docker"
    shell.privileged = false
  end
end
