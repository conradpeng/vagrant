# box

## 安装 box

- ### 官方下载<br>
    [官方box的下载地址](https://app.vagrantup.com/boxes/search)<br>

    查看要下载的 `box` 的名称，加在指令后
    ```
    vagrant box add centos/7
    ```


- ### 镜像下载<br>
    [镜像的下载地址](http://www.vagrantbox.es/)<br>
    使用镜像需要本地下载（迅雷挺快），再通过选择本地文件的方式添加`box`

    添加本地box
    ``` sh
    # 本地下载
    vagrant box add centos/7 D:/centos-7.0-x86_64.box

    # 也可以http下载
    vagrant box add centos/7 https://github.com/tommy-muehle/puppet-vagrant-boxes/releases/download/1.1.0/centos-7.0-x86_64.boxx
    ```

## 管理 box

- ### 查看 `box` 列表

    ``` sh
    # 查看box列表
    vagrant box list
    ```

- ### 删除 `box`

    版本不唯一时，后面带上版本号
    ``` sh
    vagrant box remove laravel/homestead --box-version 0
    ```

<br>

# 创建虚拟机

## 虚拟机配置文件 `vagrantFile`

- ### 生成 `vagrantFile`

    进入你建立的存放虚拟机管理文件的目录下，执行以下命令生成 `vagrantFile`
    ``` sh
    vagrant init
    ```

- ### `vagrantFile` 分析

    `vagrantFile` 使用的是 `ruby` 语法

    ``` ruby
    Vagrant.configure("2") do |config|

    # 虚拟机使用的box
    config.vm.box = "centos/7"

    # 定义box是否自动查找更新
    config.vm.box_check_update = false

    # 定义端口转发 下例中：宿主机8080转发到虚拟机的80端口
    config.vm.network "forwarded_port", guest: 80, host: 8080
    config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

    # 定义私有网络，该地址是私有网络中虚拟机的地址，我们的宿主机ip则是该私有网络的网关
    config.vm.network "private_network", ip: "192.168.33.10"

    # 定义公有网络，将虚拟机和宿主机放置于同一网络环境中，可以直接访问外部网络
    config.vm.network "public_network"

    # 目录映射，将宿主机的data目录映射到虚拟机里的vagrant_data目录
    config.vm.synced_folder "../data", "/vagrant_data"

    # 配置单元 provider指虚拟机的提供程序 virtualbox vmware等
    config.vm.provider "virtualbox" do |vb|
      # 启动时是否显示vortualbox的ui界面
      vb.gui = true
    
      # 虚拟机内存
      vb.memory = "1024"
    end

    # 设置器，仅会在虚拟机第一次启动的时候，执行以下的shell命令，用来配置该虚拟机环境，修改配置文件，安装软件等
    config.vm.provision "shell", inline: <<-SHELL
      apt-get update
      apt-get install -y apache2
    SHELL
    end
    ```

## vagrant 指令

- ### 启动
    `vagrantFile` 中的所有配置会在第一次启动的时候全部加载，但是设置器中的内容，在后续启动时就不会再加载了，此时需要使用 `--provision`参数
    ``` sh
    # 启动虚拟机
    vagrant up

    # 启动并加载设置器
    vagrant up --provision
    ```

- ### 关闭
    ``` sh
    vagrant halt
    ```

- ### 重启

    `reload` 重启时会重新加载 `vagrantFile` 中的修改过的设置，但是设置器部分的修改不会重新加载
    ``` sh
    # 重启虚拟机，会重新加载vagrantFile中的修改过的设置
    vagrant reload

    # 重启并重新加载设置器
    vagrant reload --provision
    ```

- ### 休眠
    ```
    vagrant suspend
    ```
- ### 恢复休眠

    ```
    vagrant resume
    ```


# 虚拟机管理

## box管理

- ### 打包
    ``` sh
    # 查看帮助命令
    vagrant package --help

    Options:

        --base NAME                  Name of a VM in VirtualBox to package as a base box (VirtualBox Only)
        --output NAME                Name of the file to output
        --include FILE,FILE..        Comma separated additional files to package with the box
        --vagrantfile FILE           Vagrantfile to package with the box
    -h, --help                       Print this help
    ```

    >`--base NAME` vbox里的虚拟机的名称<br>
    `--output NAME` 指要打包的box名称，需要手动添加后缀.box<br>
    `--include FILE...` 打包时包含的文件名<br>
    `--vagrantfile FILE` 打包时包含的Vagrantfile文件似<br>

    ``` sh
    vagrant package --base centos7.2_laradoc --output centos7.2_kernel-3.10.0-1160.box --vagrantfile ./vagrantFile
    ```

- ### 版本管理
    在box的目录下新建文件 `metadata.json`
    ``` json
    {
        "name": "centos/7.2",
        "versions": [{
            "version": "3.10.0-1160",
            "providers": [{
                "name": "virtualbox",
                "url": "./centos7.2_kernel-3.10.0-1160.box"
            }]
        }]
    }
    ```
    >`name`: 添加的box名字<br>
    `version`：版本号<br>
    `providers.name`：虚拟主机类型<br>
    `providers.url`：box地址

    添加box时，添加该json文件
    ``` sh
    vagrant box add metadata.json
    ```

    查看box列表就会看到带有版本号的box
    ``` sh
    $ vagrant box list
    centos/7     (virtualbox, 0)
    centos/7.2   (virtualbox, 0)
    centos/7.2   (virtualbox, 3.10.0-1160)
    lc/homestead (virtualbox, 8.2.1)
    ```

    在 `.vagrant.d\boxes\centos-VAGRANTSLASH-7.2` 目录下会出现两个不同版本号的文件夹