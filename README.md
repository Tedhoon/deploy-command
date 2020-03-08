# DEPLOY-COMMAND <django>

## apt-get essential
```bash
sudo apt-get update
sudo apt-get install build-essential
sudo apt-get install python3
sudo apt-get install python3-pip
sudo pip3 install --upgrade pip
```

## clone with ssh
```bash
ssh-keygen -t rsa
cat /home/ubuntu/.ssh/id_rsa.pub
git clone ssh
sudo apt-get install virtualenv
cd /경로
virtualenv -p python3 venv
source venv/bin/activate
pip install -r requirements.txt
```

## runserver Test
```bash
exit 해도 유지 : nohup python3 manage.py 0.0.0.0:8000 &
PID확인 : ps -ef
종료 : kill -9 [PID]
```

## uwsgi
```bash
pip install uwsgi 
vi uwsgi.ini
```

```comment
uwsgi는 manage.py 같은 경로 기준
venv는 프로젝트 폴더/venv 기준
```

```ini
[uwsgi]
chdir=/home/ubuntu/{프로젝트 폴더}
module={프로젝트 내 파일이름}.wsgi:application
master=True
pidfile=/tmp/project-master.pid
vacuum=True
max-requests=5000
daemonize=/home/ubuntu/{프로젝트 폴더}/django.log
home=/home/ubuntu/{프로젝트 폴더}/venv
virtualenv=/home/ubuntu/{프로젝트 폴더}/venv
socket=/home/ubuntu/{프로젝트 폴더}/uwsgi.sock
chmod-socket=666

[uwsgi-example]
chdir=/home/ubuntu/SPNU_DP
module=TPNU.wsgi:application
master=True
pidfile=/tmp/project-master.pid
vacuum=True
max-requests=5000
daemonize=/home/ubuntu/SPNU_DP/django.log
home=/home/ubuntu/SPNU_DP/venv
virtualenv=/home/ubuntu/SPNU_DP/venv
socket=/home/ubuntu/SPNU_DP/uwsgi.sock
chmod-socket=666
```
## uwsgi command
```python
# django-nginx연결 업데이트
uwsgi --ini uwsgi.ini
```

## nginx
### *nginx.conf, site-enabled, site-available은 서로를 참조함*
```shell
sudo apt-get install nginx
```

### nginx.conf 설정
```python
sudo vi /etc/nginx/nginx.conf

http {
	upstream django {
        server unix:/home/ubuntu/프로젝트 폴더/uwsgi.sock;
        #.sock은 uwsgi.ini와 같은 경로
		# ex) server unix:home/ubuntu/dinga6a/dingaproject/uwsgi.sock;
	}
	##
	#client_max_body_size 
    #server_name(도메인) 설정 가능 => ip접근 시 raise 400
    ...
```



### uwsgi pass 및 static, media루트 설정
```python
sudo vi etc/nginx/site-enabled/default
	location / {
		#
		#
		include /etc/nginx/uwsgi_params;
		uwsgi_pass django;
	}
	location /dingastatic/ {
		alias /home/ubuntu/dinga6a/dingaproject/dingastatic/;
	}
	location /media/ {
		alias https://dingastorage.s3.amazonaws.com/media/;
	}
    ...
```


### tip : nginx에서 유효하지 않은 응답 넣어주기
```python
server {
    listen 80 default_server;
    return 444;
}
```



## nginx Command 
```shell
sudo service nginx start
sudo service nginx stop
sudo service nginx reload
sudo service nginx restart
```



# postgresql
## install
```bash
sudo apt-get update
sudo apt-get install libpq-dev
sudo apt-get install python3-psycopg2
sudo apt-get install postgresql
```
## psql cli command
```bash
#관리자 계정 접근
sudo -u postgres psql

# root계정 비밀번호 설정
postgres=# ALTER USER postgres WITH ENCRYPTED PASSWORD 'password'
;

# user 생성
CREATE USER [username] WITH ENCRYPTED PASSWORD 'password';

# user 소유의 DB생성
CREATE DATABASE [db_name] OWNER [username];

# user에게 DB모든 권한 부여
GRANT ALL PRIVILEGES ON DATABASE [db_name] TO [username];

# 재시작
sudo /etc/init.d/postgresql restart
```

### psql
```bash
# 데이터베이스 리스트 보기
\l

# 데이터베이스 접근
\c db_name
```


### 수정 및 배포
```bash
uwsgi --ini uwsgi.ini
sudo service nginx reload
```
