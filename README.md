# gitlab-gitlab-runner
Process of Installing gitlab and gitlab-runner

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
