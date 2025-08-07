## 실행 시 필수 옵션  
Fail2Ban의 iptables 연동을 위해 반드시 아래 옵션이 필요함:  

- cap_add: NET_ADMIN
- cap_add: NET_RAW
- container_name 지정
- 포트 매핑 (-p 80:80)

docker run 사용시:  
docker run -d --name nginx-naxsi-fail2ban --cap-add=NET_ADMIN --cap-add=NET_RAW -p 80:80 nginx-naxsi-fail2ban:latest