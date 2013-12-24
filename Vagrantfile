# -*- mode: ruby -*-
# vi: set ft=ruby :

%w(vagrant-berkshelf vagrant-butcher vagrant-omnibus).each do |plugin|
  Vagrant.require_plugin plugin
end

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "precise-server-cloudimg-amd64"
  config.vm.box_url = "http://cloud-images.ubuntu.com/vagrant/precise/current/precise-server-cloudimg-amd64-vagrant-disk1.box"

  config.vm.network :forwarded_port, guest: 5000, host: 5000

  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--memory", "1024"]
  end

  config.vm.provision :shell do |shell|
    shell.inline = <<-EOS
      set -e

      if [ ! -f /var/lock/provision ]; then
        export DEBIAN_FRONTEND=noninteractive
        debconf-set-selections <<< 'mysql-server mysql-server/root_password password'
        debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password'

        apt-get install -y -q python python-pip python-dev mysql-server libmysqlclient-dev
        wget -qO- https://toolbelt.heroku.com/install-ubuntu.sh | sh

        pip install virtualenv
        cd /vagrant
        rm -rf venv
        virtualenv venv
        source venv/bin/activate
        pip install -r requirements.txt

        mysqladmin create anyway
        export CLEARDB_DATABASE_URL=mysql://localhost/anyway
        python models.py # create the DB schema
        python process.py # load the CSV into the DB

        touch /var/lock/provision # mark provisioning as completed
      fi
    EOS
  end

  config.vm.provision :shell do |shell|
    shell.inline = <<-EOS
      [ -f /var/lock/provision ] || exit 1

      cd /vagrant
      source venv/bin/activate
      export CLEARDB_DATABASE_URL=mysql://localhost/anyway
      killall gunicorn 2>/dev/null
      foreman start
    EOS
  end
end
