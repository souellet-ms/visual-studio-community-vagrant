Vagrant.configure(2) do |config|
  config.vm.box = "windows-2016-amd64"
  config.vm.provider "virtualbox" do |vb|
    vb.linked_clone = true
    vb.memory = 4096
    vb.customize ["modifyvm", :id, "--vram", 64]
    vb.customize ["modifyvm", :id, "--clipboard", "bidirectional"]
    vb.customize ["modifyvm", :id, "--draganddrop", "bidirectional"]
    vb.customize [
      "storageattach", :id,
      "--storagectl", "IDE Controller",
      "--device", 0,
      "--port", 1,
      "--type", "dvddrive",
      "--medium", "emptydrive"]
    vb.customize ["modifyvm", :id, "--usbxhci", "on"]
    audio_driver = case RUBY_PLATFORM
      when /linux/
        "alsa"
      when /darwin/
        "coreaudio"
      when /mswin|mingw|cygwin/
        "dsound"
      else
        raise "Unknown RUBY_PLATFORM=#{RUBY_PLATFORM}"
      end
    vb.customize ["modifyvm", :id, "--audio", audio_driver, "--audiocontroller", "hda"]
  end
  config.trigger.before :up do
    if @machine.id
      info "Clearing any previously set USB filters..."
      until `VBoxManage showvminfo #{@machine.id} --machinereadable | grep USBFilterName`.empty?
        run "VBoxManage usbfilter remove 0 --target #{@machine.id}"
      end
    end
  end
  config.trigger.after :up do
    info "Adding USB filters..."
    # Add a filter for a Samsung mobile phone.
    # NB run VBoxManage list usbhost to known your device details. 
    # NB the ADB driver is shipped with Samsung Kies.
    # see https://www.virtualbox.org/manual/ch03.html#settings-usb
    # see https://www.virtualbox.org/manual/ch08.html#idm5501
    run "VBoxManage usbfilter add 0 --target #{@machine.id}" +
          " --name \"Samsung Galaxy J5 (SM-J500FN)\"" +
          " --manufacturer SAMSUNG" +
          " --product SAMSUNG_Android"
  end
  config.vm.provision "shell", path: "ps.ps1", args: "provision-choco.ps1"
  config.vm.provision "shell", path: "ps.ps1", args: "provision.ps1"
  config.vm.provision "shell", path: "ps.ps1", args: "provision-dotnet.ps1"
  config.vm.provision :reload
  config.vm.provision "shell", path: "ps.ps1", args: "provision-vs.ps1"
  config.vm.provision "shell", path: "ps.ps1", args: "provision-dotnetcore-sdk.ps1"
end
