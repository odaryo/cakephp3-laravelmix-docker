# DockerをVagrant + VirtualBoxで動かしたときのTips

## Vagrantfileについて

```sh
Vagrant.configure("2") do |config|

  config.vm.define "barge"
  config.vm.box = "ailispaw/barge"
  
  config.vm.network "forwarded_port", guest: 2375, host: 2375, host_ip: "127.0.0.1"
  config.vm.network "private_network", ip: "192.168.33.10"
  config.vm.synced_folder ".", "/vagrant",type:"virtualbox",:mount_options => ['dmode=755', 'fmode=644']
  config.vm.synced_folder "D:/workspace/htdocs-cake3", "/app",type:"virtualbox",:mount_options => ['dmode=755', 'fmode=755']

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "512"
    vb.cpus = "2"
  end
  
  # sshキーを登録
  config.vm.provision "shell", inline: <<-SHELL
    cat /vagrant/id_rsa_vm.pub >> /home/bargee/.ssh/authorized_keys
  SHELL
  
  # dockerのバージョンを更新する
  config.vm.provision "shell", inline: <<-SHELL
    sudo /etc/init.d/docker restart v18.09.3
  SHELL
  
  # docker-composeのインストール（インストール先を/opt/binに設定）
  config.vm.provision "shell", inline: <<-SHELL
    wget -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-`uname -s`-`uname -m`"
    sudo mv docker-compose-`uname -s`-`uname -m` /opt/bin/docker-compose
    sudo chmod +x /opt/bin/docker-compose
  SHELL
  
  config.vm.network :forwarded_port, guest: 8080, host: 8080
  config.vm.network :forwarded_port, guest: 80, host: 80      # Web
  config.vm.network :forwarded_port, guest: 443, host: 443      # Web
  config.vm.network :forwarded_port, guest: 3306, host: 3306  # MySQL
  config.vm.network :forwarded_port, guest: 22, host: 22  # ssh
  
end
```

## 注意点
- ファイル共有時のパーミッションを755に設定する
  - ファイルに実行権限がないと、nodeでのコンパイルでエラーが出るため
- 