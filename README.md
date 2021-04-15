# docker-compose  




1. install. 
docker desktop install ( to be included compose , engine ). 

mkdir composetest  

user# touch app.py  
user# vi app.py // app.py 에 아래 내용을 저장합니다.  

---  
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
---  
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




참조 : https://docs.docker.com/compose/gettingstarted/ 
