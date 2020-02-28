

# MinIO 
- https://min.io/
- https://docs.min.io/
- Open Source, S3 Compatible, Enterprise Hardened and Really, Really Fast
	-  S3 Compatible : Client(mc), SDK (Java, Javascript, Python, Golang, .Net ..)
- high performance, distributed object storage system
- Private cloud object storage


## Getting Started
- MinID 는 Golang으로 제작되어 의존성 없는 단일 파일로 운영 가능 
- Docker의 경우 Alpine linux 로 배포 
- Downloads : https://min.io/download

### Quickstart Server 
- https://docs.min.io/docs/minio-quickstart-guide.html

```sh
# linux - server run
$ wget https://dl.min.io/server/minio/release/linux-amd64/minio
$ chmod +x minio

$ export MINIO_ACCESS_KEY=infra
$ export MINIO_SECRET_KEY=infrapass

# ./minio server --address 0.0.0.0:9000 /data
$ ./minio server /data

Status:         1 Online, 0 Offline.
Endpoint:  http://192.168.144.5:9000  http://127.0.0.1:9000

Browser Access:
   http://192.168.144.5:9000  http://127.0.0.1:9000

Object API (Amazon S3 compatible):
   Go:         https://docs.min.io/docs/golang-client-quickstart-guide
   Java:       https://docs.min.io/docs/java-client-quickstart-guide
   Python:     https://docs.min.io/docs/python-client-quickstart-guide
   JavaScript: https://docs.min.io/docs/javascript-client-quickstart-guide
   .NET:       https://docs.min.io/docs/dotnet-client-quickstart-guide
```


- docker-compose 
	- [docker-compose.yml](docker-compose/docker-compose.yml)

```yml
version: '3'
services:
  minio:
    image: minio/minio
    command: server /data
    container_name: minio
    environment:
      MINIO_ACCESS_KEY: infra
      MINIO_SECRET_KEY: infrapass
    restart: always
    ports:
      - "9000:9000"
    volumes:
      - ./data:/data
```

```sh
$ cd docker-compose

$ docker-compose up -d 
```

### Quickstart Client
- https://docs.min.io/docs/minio-client-quickstart-guide.html

```sh
# linux 
$ wget https://dl.min.io/client/mc/release/linux-amd64/mc
$ chmod +x mc
$ ./mc --help

# add server config 
# mc config host add <ALIAS> <YOUR-S3-ENDPOINT> <YOUR-ACCESS-KEY> <YOUR-SECRET-KEY>
$ ./mc config host add myminio http://centos:9000 infra infrapass
Added `myminio` successfully.

# creates a new bucket
$ ./mc mb myminio/data
Bucket created successfully ` myminio/data`.

# config.json
# {
# 	"version": "9",
# 	"hosts": {
# 		"myminio": {
# 			"url": "http://centos:9000",
# 			"accessKey": "infra",
# 			"secretKey": "infrapass",
# 			"api": "s3v4",
# 			"lookup": "auto"
# 		},
# 		...

# copy
$ ./mc cp seq-1.txt myminio/data
seq-1.txt:                 27 B / 27 B [==========================================s 0s

# list 
$ ./mc ls myminio/data
[2020-02-28 23:10:41 KST]     27B seq-1.txt

# remove 
$ ./mc rm --recursive --force myminio/data/
```

### Server Files 형식 
- .minio.sys : meta data 
- seq-1.txt : 파일이 아닌 디렉토리(Object) 형태로 저장 

```
tree

├── minio1                                       
│   ├── .minio.sys                               
│   │   ├── backend-encrypted                    
│   │   │   ├── part.1                           
│   │   │   └── xl.json                          
│   │   ├── background-ops                       
│   │   │   └── data-usage                       
│   │   │       ├── part.1                       
│   │   │       └── xl.json                      
│   │   ├── config                               
│   │   │   ├── config.json                      
│   │   │   │   ├── part.1                       
│   │   │   │   └── xl.json                      
│   │   │   └── iam                              
│   │   │       └── format.json                  
│   │   │           ├── part.1                   
│   │   │           └── xl.json                  
│   │   ├── format.json                          
│   │   ├── multipart                            
│   │   └── tmp                                  
│   └── data                                     
│       └── seq-1.txt                            
│           ├── part.1                           
│           └── xl.json                          
```

## MinIO Erasure Code
- https://docs.min.io/docs/minio-erasure-code-quickstart-guide.html

### Erasure Code 
- 누락되거나 손상된 데이터를 재구성하는 수학적 알고리즘
- Erasure code와 Checksums 사용하여 하드웨어 오류 및 자동 데이터 손상으로부터 데이터를 보호
- 중복 수준이 높으면 전체 드라이브의 최대 절반 (N/2)이 손실 되어도 데이터를 복구 가능 
- 드라이브를 4, 6, 8, 10, 12, 14 또는 16 개의 erasure-coding sets 구성 (최소 4개)

### Run MinIO Server with Erasure Code
- 4 drives setup
- Drives를 물리적인 디스크로 구성하면 별도의 Raid 구성이 필요 없음 

```sh
$ minio server /data1 /data2 /data3 /data4
```

#### docker-compose 
- [docker-compose.yml](docker-compose/erasure-code/docker-compose.yml)

```yml
version: '3'
services:
  minio1:
    image: minio/minio
    command: server /data1 /data2 /data3 /data4
    container_name: minio1
    environment:
      MINIO_ACCESS_KEY: infra
      MINIO_SECRET_KEY: infrapass
    restart: always
    ports:
      - "9000:9000"
    volumes:
      - ./data1:/data1
      - ./data2:/data2
      - ./data3:/data3
      - ./data4:/data4
```


## Distributed MinIO 
- https://docs.min.io/docs/distributed-minio-quickstart-guide.html
- MinIO in distributed mode lets you pool multiple drives (even on different machines) into a single object storage server.
- 서버를 분리하여 다중 Drives를 지원
	- Data protection
	- High availability
	- Consistency Guarantees


### Run distributed MinIO
- MINIO_ACCESS_KEY and MINIO_SECRET_KEY 같은 키로 구성 
- Erasure Code와 동일한 Drivers 정책으로 분산 
- Distributed MinIO 와 서버별로 Erasure Code 같이 적용 가능 



#### docker-compose : docker로 4대 서버 시뮬레이션 
- [docker-compose.yml](docker-compose/distributed/docker-compose.yml)

```yml
version: '3'
services:
  minio1:
    image: minio/minio
    command: server http://minio{1...4}:9000/data
    container_name: minio1
    environment:
      MINIO_ACCESS_KEY: infra
      MINIO_SECRET_KEY: infrapass
    restart: always
    ports:
      - "9001:9000"
    volumes:
      - ./minio1:/data

  minio2:
    image: minio/minio
    command: server http://minio{1...4}:9000/data
    container_name: minio2
    environment:
      MINIO_ACCESS_KEY: infra
      MINIO_SECRET_KEY: infrapass
    restart: always
    ports:
      - "9002:9000"
    volumes:
      - ./minio2:/data

  minio3:
    image: minio/minio
    command: server http://minio{1...4}:9000/data
    container_name: minio3
    environment:
      MINIO_ACCESS_KEY: infra
      MINIO_SECRET_KEY: infrapass
    restart: always
    ports:
      - "9003:9000"
    volumes:
      - ./minio3:/data

  minio4:
    image: minio/minio
    command: server http://minio{1...4}:9000/data
    container_name: minio4
    environment:
      MINIO_ACCESS_KEY: infra
      MINIO_SECRET_KEY: infrapass
    restart: always
    ports:
      - "9004:9000"
    volumes:
	  - ./minio4:/data
```


## SDK - Python 
- https://docs.min.io/docs/python-client-quickstart-guide.html
- https://docs.min.io/docs/python-client-api-reference.html
