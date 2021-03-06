require 'yaml'
require 'fileutils'


config = {
  local: 'vagrant/config/vagrant-local.yml',
  example: 'vagrant/config/vagrant-local.example.yml'
}

# copy config from example if local config not exists
FileUtils.cp config[:example], config[:local] unless File.exist?(config[:local])
# read config
options = YAML.load_file config[:local]

options['username'] = 'ubuntu'
options['group'] = 'ubuntu'

# check github token
if options['github_token'].nil? || options['github_token'].to_s.length != 40
  puts "You must place REAL GitHub token into configuration:/vagrant/config/vagrant-local.yml"
  exit
end

# vagrant configurate
Vagrant.configure(2) do |config|
  config.vm.box = 'ubuntu/xenial64'

  # should we ask about box updates?
  config.vm.box_check_update = options['box_check_update']

  config.vm.provider 'virtualbox' do |vb|
    # machine cpus count
    vb.cpus = options['cpus']
    # machine memory size
    vb.memory = options['memory']
    # machine name (for VirtualBox UI)
    vb.name = options['machine_name']
  end

  # machine name (for vagrant console)
  config.vm.define options['machine_name']
  
  # machine name (for guest machine console)
  config.vm.hostname = options['machine_name']
  
  # network settings
  config.vm.network 'private_network', ip: options['ip']
  
  # sync: folder  (host machine) -> folder '/vagrant' (guest machine)
  config.vm.synced_folder './', '/vagrant', owner: options['username'], group: options['group']

  # hosts settings (host machine)
  config.vm.provision :hostmanager
  config.hostmanager.enabled            = true
  config.hostmanager.manage_host        = true
  config.hostmanager.ignore_private_ip  = false
  config.hostmanager.include_offline    = true
  config.hostmanager.aliases            = [options['domain']]

  # provisioners
  config.vm.post_up_message = "Provisioning environment http://#{options['domain']}"
  config.vm.provision 'shell', path: 'vagrant/install.sh', args: [options['timezone']]
  config.vm.provision 'shell', path: 'vagrant/setup.sh'
  config.vm.provision 'shell', path: 'vagrant/build.sh', run: 'always'
end