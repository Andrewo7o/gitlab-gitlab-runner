
preparation：安装docker并配置好加速器

1.
docker run \  
       --publish 443:443 --publish 10080:80 --publish 2222:22 \
       --name gitlab \
       --restart always \
       --volume /srv/gitlab/config: /etc/gitlab \    
       --volume /srv/gitlab/logs: /var/log/gitlab \    
       --volume /srv/gitlab/data: /var/opt/gitlab \    
       gitlab/gitlab-ce:latest

三类端口映射：
             端口：22   服务：SSH     说明：SSH通过在网络中建立安全隧道来实现SSH客户端与服务器之间的连接
             端口：80   服务：HTTP    说明：用于网页浏览
             端口：443  服务：Https   说明：网页浏览端口，能提供加密和通过安全端口传输的另一种HTTP

三个挂载目录：
             /etc/gitlab
             /var/log/gitlab
             /var/opt/gitlab

2.
在gitlab上创建项目时，生成项目的URL访问地址是按容器的hostname来生成的，即容器的id
   进入容器内   vim /etc/gitlab/gitlab.rb
      a.external_url "宿主机IP+映射端口"
      b.#nginx配置   nginx["listen_port"] = "80"
3.
启动gitlab-runner，默认root用户
      docker run  -d -v /srv/gitlab-runner/config:/etc/gitlab-runner -v /srv/gitlab-runner/home:/home/gitlab-runner 
      --restart always --name gitlab-runner gitlab/gitlab-runner:latest 
      run --user=root --working-directory=/home/gitlab-runner

4.
注册gitlab runner，executor为shell
      docker exec -it gitlab-runner gitlab-runner register --run-untagged="true"  --locked="false"



注意事项：
       1. gitlab runner 注册时要加命令参数 
               --run-untagged = "true"
               --locked = "false"
           否则会出现Jobs stuck问题，没有runner来执行yml文件
       2. 启动gitlab runner时，通过sudo提升gitlab-runner用户权限，虽然可以创建config文件，但无法创建数据文件
               sudo docker run  -d -v /srv/gitlab-runner/config:/etc/gitlab-runner -v /srv/gitlab-runner/home:/home/gitlab-runner 
                        --restart always --name gitlab-runner gitlab/gitlab-runner:latest
           解决方法：默认是root用户
       3. 启动gitlab-runner+debug模式
               docker run  -d -v /srv/gitlab-runner/config:/etc/gitlab-runner -v /srv/gitlab-runner/home:/home/gitlab-runner 
               --restart always    -e DEBUG=true    --name gitlab-runner gitlab/gitlab-runner:latest  
               run --user=root   --working-directory=/home/gitlab-runner
       4.  启动gitlab-runner，executor为shell
       5.  每次注册新的runner之前，要把之前废掉的runner的注册信息删掉
                  注册信息所在位置：/srv/gitlab-runner/config/config.toml
       6. 如果要重新启动一个gitlab容器，应该先前把挂载出的目录清理干净，否则会出现读取数据库错误
       7. 若在注册gitlab runner时出现URL错误
              a. 关闭防火墙
                    systemctl status firewalld
                    systemctl stop firewalld
                    systemctl disable firewalled
              b. 关闭SELinux
                    临时关闭：setenforce 0
                    修改配置文件： /etc/selinux/config文件  将SELINUX=enforcing 改为 SELINUX=disabled
       
             
补充(测试过程)：
      1. git上传文件
           git init
           git clone http://10.10.26.85:10080/root/test.git
           git add .
           git commit -m "update yml"
           git push
      2. yml文件 -- .gitlab-ci.yml
           stages:
              - build
              - test
           job1:
              stage: test
              script: 
                 - echo "I am job1"
                 - echo "I am in testing stage"
           job2:
              stage: build
              script:
                 - echo "I am job2"
                 - echo " I am in building stage"

杂记：
    1. 镜像打包
         docker save -o 镜像名称.tar 指定镜像:标签
           或 docker save 镜像ID < 镜像名称.tar
    2. 镜像解压
         docker load < 镜像名称.tar
         docker tag 镜像ID 镜像名称:标签
    3. docker内安装ping:
         a. apt-get update
         b. apt-get install iputils-ping
    4. 进入运行中的容器
         docker exec -it 镜像名称 bash
    5. 查看容器的运行日志
         docker logs 容器名称 > log
