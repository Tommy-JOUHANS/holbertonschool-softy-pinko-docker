task4:

(venv) tommy@TommyJOUHANSPRO:~/holbertonschool-softy-pinko-docker/task4$ docker-compose down  
[+] down 3/3
 ✔ Container task4-front-end-1 Removed                                                                                                               0.1s
 ✔ Container task4-back-end-1  Removed                                                                                                               0.1s
 ✔ Network task4_default       Removed                                                                                                               1.2s
(venv) tommy@TommyJOUHANSPRO:~/holbertonschool-softy-pinko-docker/task4$ docker-compose build --no-cache
[+] Building 58.3s (24/24) FINISHED                                                                                                                      
 => [internal] load local bake definitions                                                                                                          0.0s
 => => reading from stdin 1.15kB                                                                                                                    0.0s
 => [front-end internal] load build definition from Dockerfile                                                                                      0.0s
 => => transferring dockerfile: 472B                                                                                                                0.0s
 => [back-end internal] load build definition from Dockerfile                                                                                       0.0s
 => => transferring dockerfile: 322B                                                                                                                0.0s
 => [back-end internal] load metadata for docker.io/library/ubuntu:24.04                                                                            1.4s
 => [front-end internal] load metadata for docker.io/library/nginx:latest                                                                           0.0s
 => [front-end internal] load .dockerignore                                                                                                         0.0s
 => => transferring context: 2B                                                                                                                     0.0s
 => CACHED [front-end 1/3] FROM docker.io/library/nginx:latest                                                                                      0.0s
 => [front-end internal] load build context                                                                                                         0.0s
 => => transferring context: 6.43kB                                                                                                                 0.0s
 => [front-end 2/3] COPY ./softy-pinko-front-end /var/www/html/softy-pinko-front-end                                                                0.2s
 => [front-end 3/3] COPY ./softy-pinko-front-end.conf /etc/nginx/conf.d/default.conf                                                                0.1s
 => [front-end] exporting to image                                                                                                                  0.2s
 => => exporting layers                                                                                                                             0.1s
 => => writing image sha256:5973e248d60d8227d72632f733c408a7ce8bf2b834d2799f322b071f52e668f7                                                        0.0s
 => => naming to docker.io/library/softy-pinko-front-end:task4                                                                                      0.0s
 => [auth] library/ubuntu:pull token for registry-1.docker.io                                                                                       0.0s
 => [front-end] resolving provenance for metadata file                                                                                              0.0s
 => [back-end internal] load .dockerignore                                                                                                          0.0s
 => => transferring context: 2B                                                                                                                     0.0s
 => CACHED [back-end 1/7] FROM docker.io/library/ubuntu:24.04@sha256:c4a8d5503dfb2a3eb8ab5f807da5bc69a85730fb49b5cfca2330194ebcc41c7b               0.0s
 => [back-end internal] load build context                                                                                                          0.0s
 => => transferring context: 63B                                                                                                                    0.0s
 => [back-end 2/7] RUN apt-get update && apt-get install -y python3 python3-pip                                                                    48.2s
 => [back-end 3/7] RUN rm /usr/lib/python*/EXTERNALLY-MANAGED || true                                                                               0.4s
 => [back-end 4/7] WORKDIR /app                                                                                                                     0.1s
 => [back-end 5/7] COPY requirements.txt /app                                                                                                       0.1s
 => [back-end 6/7] RUN pip install -r /app/requirements.txt                                                                                         3.3s
 => [back-end 7/7] COPY api.py /app                                                                                                                 0.1s
 => [back-end] exporting to image                                                                                                                   3.6s
 => => exporting layers                                                                                                                             3.5s
 => => writing image sha256:2c377f83993d38d4324248686f895cf63ae72f33a78d6431852d88c55d3063e9                                                        0.0s
 => => naming to docker.io/library/softy-pinko-back-end:task4                                                                                       0.0s
 => [back-end] resolving provenance for metadata file                                                                                               0.0s
[+] build 2/2
 ✔ Image softy-pinko-back-end:task4  Built                                                                                                          61.1s
 ✔ Image softy-pinko-front-end:task4 Built                                                                                                          61.1s
(venv) tommy@TommyJOUHANSPRO:~/holbertonschool-softy-pinko-docker/task4$ docker-compose up
[+] up 3/3
 ✔ Network task4_default       Created                                                                                                               1.3s
 ✔ Container task4-back-end-1  Created                                                                                                               0.1s
 ✔ Container task4-front-end-1 Created                                                                                                               0.1s
Attaching to back-end-1, front-end-1
back-end-1  |  * Serving Flask app 'api'
back-end-1  |  * Debug mode: off
back-end-1  | WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
back-end-1  |  * Running on all addresses (0.0.0.0)
back-end-1  |  * Running on http://127.0.0.1:5252
back-end-1  |  * Running on http://172.19.0.2:5252
back-end-1  | Press CTRL+C to quit
front-end-1  | /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
front-end-1  | /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
front-end-1  | /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
front-end-1  | 10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
front-end-1  | 10-listen-on-ipv6-by-default.sh: info: /etc/nginx/conf.d/default.conf differs from the packaged version
front-end-1  | /docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
front-end-1  | /docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
front-end-1  | /docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
front-end-1  | /docker-entrypoint.sh: Configuration complete; ready for start up
front-end-1  | 2026/05/18 14:45:45 [notice] 1#1: using the "epoll" event method
front-end-1  | 2026/05/18 14:45:45 [notice] 1#1: nginx/1.31.0
front-end-1  | 2026/05/18 14:45:45 [notice] 1#1: built by gcc 14.2.0 (Debian 14.2.0-19) 
front-end-1  | 2026/05/18 14:45:45 [notice] 1#1: OS: Linux 6.6.87.2-microsoft-standard-WSL2
front-end-1  | 2026/05/18 14:45:45 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1024:1048576
front-end-1  | 2026/05/18 14:45:45 [notice] 1#1: start worker processes
front-end-1  | 2026/05/18 14:45:45 [notice] 1#1: start worker process 28
front-end-1  | 2026/05/18 14:45:45 [notice] 1#1: start worker process 29
front-end-1  | 2026/05/18 14:45:45 [notice] 1#1: start worker process 30
front-end-1  | 2026/05/18 14:45:45 [notice] 1#1: start worker process 31
front-end-1  | 2026/05/18 14:45:45 [notice] 1#1: start worker process 32
front-end-1  | 2026/05/18 14:45:45 [notice] 1#1: start worker process 33
front-end-1  | 2026/05/18 14:45:45 [notice] 1#1: start worker process 34
front-end-1  | 2026/05/18 14:45:45 [notice] 1#1: start worker process 35
front-end-1  | 2026/05/18 14:45:45 [notice] 1#1: start worker process 36
front-end-1  | 2026/05/18 14:45:45 [notice] 1#1: start worker process 37
front-end-1  | 2026/05/18 14:45:45 [notice] 1#1: start worker process 38
front-end-1  | 2026/05/18 14:45:45 [notice] 1#1: start worker process 39
front-end-1  | 2026/05/18 14:45:45 [notice] 1#1: start worker process 40
front-end-1  | 2026/05/18 14:45:45 [notice] 1#1: start worker process 41
front-end-1  | 2026/05/18 14:45:45 [notice] 1#1: start worker process 42
front-end-1  | 2026/05/18 14:45:45 [notice] 1#1: start worker process 43
back-end-1   | 172.19.0.1 - - [18/May/2026 14:45:54] "GET /api/hello HTTP/1.1" 200 -
back-end-1   | 172.19.0.1 - - [18/May/2026 14:45:54] "GET /favicon.ico HTTP/1.1" 404 -
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/css/bootstrap.min.css HTTP/1.1" 304 0 "http://localhost:9000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/css/font-awesome.css HTTP/1.1" 304 0 "http://localhost:9000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/images/left-image.png HTTP/1.1" 304 0 "http://localhost:9000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/css/templatemo-softy-pinko.css HTTP/1.1" 304 0 "http://localhost:9000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/js/popper.js HTTP/1.1" 304 0 "http://localhost:9000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/js/bootstrap.min.js HTTP/1.1" 304 0 "http://localhost:9000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/js/jquery-2.1.0.min.js HTTP/1.1" 304 0 "http://localhost:9000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/js/scrollreveal.min.js HTTP/1.1" 304 0 "http://localhost:9000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/images/featured-item-01.png HTTP/1.1" 304 0 "http://localhost:9000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/js/waypoints.min.js HTTP/1.1" 304 0 "http://localhost:9000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/js/jquery.counterup.min.js HTTP/1.1" 304 0 "http://localhost:9000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/js/imgfix.min.js HTTP/1.1" 304 0 "http://localhost:9000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/images/logo.png HTTP/1.1" 304 0 "http://localhost:9000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/js/custom.js HTTP/1.1" 304 0 "http://localhost:9000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/images/right-image.png HTTP/1.1" 304 0 "http://localhost:9000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/images/testimonial-icon.png HTTP/1.1" 304 0 "http://localhost:9000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/images/work-process-item-01.png HTTP/1.1" 304 0 "http://localhost:9000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/images/blog-item-02.png HTTP/1.1" 304 0 "http://localhost:9000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/images/blog-item-01.png HTTP/1.1" 304 0 "http://localhost:9000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/images/blog-item-03.png HTTP/1.1" 304 0 "http://localhost:9000/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/images/banner-bg.png HTTP/1.1" 304 0 "http://localhost:9000/assets/css/templatemo-softy-pinko.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/images/fun-facts-bg.png HTTP/1.1" 304 0 "http://localhost:9000/assets/css/templatemo-softy-pinko.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/images/work-process-bg.png HTTP/1.1" 304 0 "http://localhost:9000/assets/css/templatemo-softy-pinko.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
front-end-1  | 172.19.0.1 - - [18/May/2026:14:45:58 +0000] "GET /assets/fonts/fontawesome-webfont.woff2?v=4.7.0 HTTP/1.1" 304 0 "http://localhost:9000/assets/css/font-awesome.css" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/148.0.0.0 Safari/537.36" "-"
back-end-1   | 172.19.0.1 - - [18/May/2026 14:45:59] "GET /api/hello HTTP/1.1" 200 -


w Enable Watch   d Detach
Le site : http://localhost:9000
L'API : http://localhost:5252/api/hello