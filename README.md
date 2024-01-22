sudo yum update -y 
sudo yum install vim wget curl unzip -y 
wget https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.rpm
sudo yum -y install ./jdk-17_linux-x64_bin.rpm
java --version
sudo yum install git -y


Create a user for sonar:
sudo useradd sonar
sudo passwd sonar


Install and configure PostgreSQL:
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
sudo yum -y install postgresql14-server postgresql14
sudo /usr/pgsql-14/bin/postgresql-14-setup initdb
sudo systemctl enable --now postgresql-14
sudo vim /var/lib/pgsql/14/data/pg_hba.conf
/var/lib/pgsql/14/data/postgresql.conf			=>>>>> #(listen_addresses = '*')
sudo vim /var/lib/pgsql/14/data/pg_hba.conf
sudo systemctl restart postgresql-14
sudo systemctl status postgresql-14



Set PostgreSQL admin user:
sudo su - postgres
psql
alter user postgres with password 'postgresql';
exit



Create a SonarQube user and database:
createdb sonarqube
psql
CREATE USER sonarqube WITH PASSWORD 'postgresql';
GRANT ALL PRIVILEGES ON DATABASE sonarqube to sonarqube;
\q



Fetch and install SonarQube:
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.4.0.54424.zip
unzip sonarqube-9.4.0.54424.zip
sudo mv sonarqube-*/  /opt/sonarqube



sudo vi /opt/sonarqube/conf/sonar.properties
##Database details
sonar.jdbc.username=sonarqube
sonar.jdbc.password=postgresql



sudo chown -R sonar:sonar /opt/sonarqube
sudo mkdir -p /var/sonarqube
sudo chown -R sonar:sonar /var/sonarqube



sudo vi /etc/systemd/system/sonarqube.service
[Unit]
Description=SonarQube service
After=syslog.target network.target
[Service]
Type=forking
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
LimitNOFILE=65536
LimitNPROC=4096
User=sonar
Group=sonar
Restart=on-failure
[Install]
WantedBy=multi-user.target



sudo vi /opt/sonarqube/bin/linux-x86-64/sonar.sh
RUN_AS_USER=sonar



sudo systemctl daemon-reload
sudo systemctl start sonarqube.service
sudo systemctl enable sonarqube.service
systemctl status sonarqube.service



Firewall rules to allow SonarQube Access:
sudo systemctl status firewalld
sudo systemctl start firewalld
sudo systemctl status firewalld
sudo firewall-cmd --permanent --add-port=9000/tcp && sudo firewall-cmd --reload



Jenkins Install:
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
yum install fontconfig java-17-openjdk
yum install jenkins
sudo systemctl daemon-reload
sudo systemctl status firewalld
sudo systemctl start firewalld
sudo systemctl status firewalld
sudo firewall-cmd --permanent --add-port=8080/tcp && sudo firewall-cmd --reload  
systemctl start jenkins.service



Add node:


status check:
systemctl start jenkins.service systemctl start sonarqube.service
systemctl status jenkins.service systemctl status sonarqube.service
systemctl restart jenkins.service systemctl restart sonarqube.service
sudo systemctl daemon-reload sudo systemctl start sonarqube.service sudo systemctl enable sonarqube.service systemctl status sonarqube.service
