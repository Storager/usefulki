# AWX
## Installation in docker as non-root user
### Centos/Redhat 7
#### Install docker
```shell
sudo sed -i 's|SELINUX=enforcing|SELINUX=permissive|g' /etc/selinux/config
sudo yum install -y yum-utils
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce docker-ce-cli containerd.io iptables
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER # $USER is a non-privileged user who runs ansible

sudo reboot

```

#### Install dependency for AWX
```shell

sudo dnf install -f https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

sudo dnf install -y ansible git


# Change to root!
sudo -i

pip3 install docker docker-compose

# Switch to normal user
exit
````

#### Install AWX

```shell
git clone https://github.com/ansible/awx.git -b 15.0.0

cd awx/installer

ansible-playbook -i inventory install.yml

```

#### Modify AWX

```shell
cd ~/.awx/awxcompose
docker-compose down
docker volume prune -f
sed -i.bak 's|~|\$HOME|g' docker-compose.yml
sed -i.bak 's|user: root|#user: root|g' docker-compose.yml
for i in SECRET_KEY environment.sh credentials.py nginx.conf redis.conf redis_socket; do chown -R 1000 ./$i; done
```
#### Manually add user for redis between lines 57 ad 58
```yaml
    restart: unless-stopped
    user: "1000"
    environment:
```

#### Start and then stop docker-compose
```shell
docker-compose up -d && docker-compose down
```
#### Change prmissions for volumes and run again
```shell
for i in awxcompose_rsyslog-socket awxcompose_rsyslog-config awxcompose_supervisor-socket
	do sudo chown -R 1000 /var/lib/docker/volumes/$i
done
docker-compose up -d
```