# docker-compose  




1. install. 
docker desktop install ( to be included compose , engine ). 

mkdir composetest  

user# touch app.py  
user# vi app.py // app.py 에 아래 내용을 저장합니다.  


import time  

import redis  
from flask import Flask  

app = Flask(__name__)  
cache = redis.Redis(host='redis', port=6379)  

def get_hit_count():  
    retries = 5  
    while True:  
        try:  
            return cache.incr('hits')  
        except redis.exceptions.ConnectionError as exc:  
            if retries == 0:  
                raise exc  
            retries -= 1  
            time.sleep(0.5)  
  
 @app.route('/')  
 def hello():  
    count = get_hit_count()  
    return 'Hello World! I have been seen {} times.\n'.format(count)  

위 예제는 redis 애플리케이션을 사용합니다.  
기본 포트인 6379를 사용합니다.  

user# touch requirements.txt  // 안에 내용을 추가합니다.
flask  
redis  

user# touch Dockerfile // 안에 내용을 추가합니다.  
FROM python:3.7-alpine // python3.7이미지로 시작하는 python3.7버전을 사용하는 이미지를 빌드합니다.
WORKDIR /code // WORKDIR로 code를 사용합니다.  
ENV FLASK_APP=app.py //flask에 사용되는 환경 변수를 지정합니다.  
ENV FLASK_RUN_HOST=0.0.0.0 // 동작 하는 호스트를 지정합니다. 0.0.0.0은 모든 네트워크를 뜻합니다.  
RUN apk add --no-cache gcc musl-dev linux-headers // gcc 및 기타 종속성을 가진 패키지나 모듈을 설치합니다.  
COPY requirements.txt requirements.txt // python 종속성을 복사한 후 설치합니다.  
RUN pip install -r requirements.txt // requirement.txt를 사용하여 flask와 redis를 설치합니다.  
EXPOSE 5000 // 컨테이너가 포트 5000에서 수신 중인 것을 설정합니다.
COPY . . // 첫 번째 .은 현재 디렉토리를 두 번째 .은 현재 디렉토리의 workdir에 복사합니다.  
CMD ["flask", "run"] // flask의 기본 명령으로 run이 동작하도록 설정합니다.  

user# touch docker-compose.yml // 안에 내용을 추가합니다.  
version: "3.9"  
services:  
  web:  
    build: .  
    ports:  
      - "5000:5000"  
  redis:  
    image: "redis:alpine"  
    

web port : 5000으로 접속하면 5000으로 가도록 설정  
image : redis 이미지를 사용  

user# docker-compose up // compose 빌드 및 실행  
![image](https://user-images.githubusercontent.com/74689088/114859363-9c686d00-9e25-11eb-8b65-8d2d0acd5043.png)  
명령어 실행 시 화면  

web_1    |  * Serving Flask app "app.py"  
web_1    |  * Environment: production  
web_1    |    WARNING: This is a development server. Do not use it in a production deployment.  
web_1    |    Use a production WSGI server instead.  
web_1    |  * Debug mode: off  
web_1    |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)  

정상적으로 실행되었다면 localhost:5000으로 접속  
![image](https://user-images.githubusercontent.com/74689088/114859515-d3d71980-9e25-11eb-8888-61a030bd8b8d.png)  
접속 시 화면  
새로 고침 시 카운터가 증가해야 정상 구동 완료  

user# docker image ls // 구성된 도커 이미지 확인  
![image](https://user-images.githubusercontent.com/74689088/114859718-18fb4b80-9e26-11eb-95a0-d5d3517030ee.png)  
사진과 같이 docker image가 확인 가능해야함  

user# vi docker-compose.yml // yml 파일 수정하여 volumes 추가  
version: "3.9"  
services:  
  web:  
    build: .  
    ports:  
      - "5000:5000"  
    volumes:  
      - .:/code  
    environment:  
      FLASK_ENV: development  
  redis:  
    image: "redis:alpine"  
    
 volumes 추가 시 /code 컨테이너 내부에 마운트 되므로 이미지를 다시 빌드하지 않고 즉시 코드 수정이 가능  
 environment키, FLASK_ENV, flask run개발 모드에서 실행하고 변경한 코드를 다시 로드하여 사용가능  

user# docker-compuse up // 앱 재빌드 및 실행  
다시 웹에 카운트 증가를 확인  

app.py에서 return 'Hello from Docker! I have been seen {} times.\n'.format(count) 으로 내용 변경하고 저장  

![image](https://user-images.githubusercontent.com/74689088/114860169-9e7efb80-9e26-11eb-9877-f3a3f3ac1b3b.png)
이후 페이지에 적용된 것을 확인할 수 있음  


user# docker-compose up -d // 백그라운드로 실행 
user#  docker-compose ps // 실행 중인 프로세스 확인  
user# docker-compose run web env // web 서비스 환경 변수 확인  
user# docker-compose stop // compose 서비스 중지  
user# docker-compose down --volumes // down으로 다른 컨테이너를 완전히 중지하고 제거합니다. --volumes로 컨테이너가 사용하는 데이터 볼륨을 제거할 수 있습니다.  


참조 : https://docs.docker.com/compose/gettingstarted/ 
