# Install required plugins
required_plugins = ["vagrant-hostsupdater", "vagrant-berkshelf"]
required_plugins.each do |plugin|
  unless Vagrant.has_plugin?(plugin)
    # User vagrant plugin manager to install plugin, which will automatically refresh plugin list afterwards
    puts "Installing vagrant plugin #{plugin}"
    Vagrant::Plugin::Manager.instance.install_plugin plugin
    puts "Installed vagrant plugin #{plugin}"
  end
end

def set_env vars
  command = <<~HEREDOC
    echo "Setting Environment Variables"
    source ~/.bashrc
  HEREDOC
  vars.each do |key, value|
    command +=<<~HEREDOC
      if [ -z "$#{key}" ]; then
        echo "export #{key}=#{value}" >> ~/.bashrc
      fi
    HEREDOC
  end
  return command
end

Vagrant.configure("2") do |config|

  config.vm.define "app" do |app|
    app.vm.box = "ubuntu/xenial64"
    app.vm.network "private_network", ip: "192.168.10.100"
    app.hostsupdater.aliases = ["development.local"]

    # Synced app folder
    app.vm.synced_folder "app", "/home/ubuntu/app"
    app.vm.synced_folder "environment_bak/app", "/home/ubuntu/environment"

    # provision with chef
    app.vm.provision "chef_solo" do |chef|
        chef.add_recipe "node-server::default"
    end

    # app.vm.provision "shell", path: "environment_bak/app/provision.sh", privileged: false
    app.vm.provision "shell", inline: set_env({DB_HOST:"mongodb://192.168.10.150"}), privileged: false

  end

  config.vm.define "db" do |db|
    db.vm.box = "ubuntu/xenial64"
    db.vm.network "private_network", ip: "192.168.10.150"
    db.hostsupdater.aliases = ["database.local"]

    # Synced DB folder
    db.vm.synced_folder "environment_bak/db", "/home/ubuntu/environment"

    # provision with chef
    db.vm.provision "chef_solo" do |chef|
        chef.add_recipe "mongo-server::default"
    end

    # db.vm.provision "shell", path: "environment_bak/db/provision.sh", privileged: false

  end
end
