Vagrant.configure("2") do |config|
  config.vm.box = "windows10"
  config.vm.guest = :windows

  # Configure Vagrant to use WinRM instead of SSH
  config.vm.communicator = "winrm"

  # Configure WinRM Connectivity
  config.winrm.username = "IEUser"
  config.winrm.password = "Passw0rd!"

  config.vm.provider "virtualbox" do |vb|
     vb.name = "Windows10Ansible"
     # Display the VirtualBox GUI when booting the machine
     vb.gui = true
   end
end