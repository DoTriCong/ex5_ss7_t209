# Troubleshoot Port 8082 - Address Already in Use

## 1. Giả lập xung đột port 8082

Chạy lệnh để tạo tiến trình Python sử dụng port `8082`:

```bash
python3 -m http.server 8082 > /dev/null 2>&1 &
```

Khi chạy lại lệnh lần thứ hai, tiến trình không thể khởi động do port `8082` đã được sử dụng.

## 2. Kiểm tra tiến trình đang chiếm port 8082

Sử dụng lệnh:

```bash
sudo ss -tlnp | grep 8082
```

Kết quả:

```text
LISTEN 0      5            0.0.0.0:8082       0.0.0.0:*    users:(("python3",pid=1689,fd=3))
```

Xác định:

* Port: `8082`
* Process: `python3`
* PID: `1689`

Kiểm tra thông tin tiến trình:

```bash
ps -p 1689 -f
```

## 3. Giải phóng port 8082

Dừng tiến trình đang chiếm port:

```bash
kill 1689
```

Kiểm tra lại:

```bash
sudo ss -tlnp | grep 8082
```

Nếu không có kết quả, port `8082` đã được giải phóng.

## 4. Khởi động lại Spring Boot

Sau khi giải phóng port `8082`, khởi động lại service:

```bash
sudo systemctl restart spring-app.service
```

Kiểm tra trạng thái:

```bash
sudo systemctl status spring-app.service --no-pager
```

Kết quả mong đợi:

```text
Active: active (running)
```

## 5. Kiểm tra port 8082 sau khi khởi động Spring Boot

```bash
sudo ss -tlnp | grep 8082
```

Kết quả mong đợi:

```text
LISTEN ... 0.0.0.0:8082 ... users:(("java",pid=...,fd=...))
```

Port `8082` lúc này được tiến trình `java` của ứng dụng Spring Boot sử dụng.

## 6. Kết luận

Đã xác định được tiến trình `python3` với PID `1689` đang chiếm dụng port `8082`.

Sau khi sử dụng lệnh:

```bash
kill 1689
```

để kết thúc tiến trình, port `8082` được giải phóng.

Sau đó khởi động lại `spring-app.service` và kiểm tra để xác nhận port `8082` được ứng dụng Spring Boot sử dụng.


Last login: Wed Oct  7 13:55:54 2026 from 42.119.42.176
root@vps-2:~# python3 -m http.server 8082 > /dev/null 2>&1 &
[1] 22466
root@vps-2:~# python3 -m http.server 8082 > /dev/null 2>&1 &
[2] 22476
[1]   Exit 1                     python3 -m http.server 8082 > /dev/null 2>&1
root@vps-2:~# sudo ss -tlnp | grep 8082
LISTEN 0      5            0.0.0.0:8082       0.0.0.0:*    users:(("python3",pid=1689,fd=3))
[2]+  Exit 1                     python3 -m http.server 8082 > /dev/null 2>&1
root@vps-2:~# ps -p 22476 -f
UID          PID    PPID  C STIME TTY          TIME CMD
root@vps-2:~# ps -p 1689 -f
UID          PID    PPID  C STIME TTY          TIME CMD
root        1689       1  0 13:51 ?        00:00:00 python3 -m http.server 8082
root@vps-2:~# sudo ss -tlnp | grep 8082
LISTEN 0      5            0.0.0.0:8082       0.0.0.0:*    users:(("python3",pid=1689,fd=3))
root@vps-2:~# kill -9 1689
root@vps-2:~# sudo ss -tlnp | grep 8082
root@vps-2:~# sudo systemctl restart spring-app.service
root@vps-2:~# sudo systemctl status spring-app.service --no-pager
● spring-app.service - Spring Boot Application
     Loaded: loaded (/etc/systemd/system/spring-app.service; disabled; preset: enabled)
     Active: active (running) since Wed 2026-10-07 14:10:19 UTC; 4s ago
 Invocation: 02e276f0735e4be2b3c3fa17e0bd66d9
   Main PID: 22566 (java)
      Tasks: 15 (limit: 578)
     Memory: 94.3M (peak: 95.5M)
        CPU: 3.756s
     CGroup: /system.slice/spring-app.service
             └─22566 /usr/bin/java -jar /opt/spring-app/app.jar

Oct 07 14:10:19 vps-2 systemd[1]: Started spring-app.service - Spring Boot Application.
root@vps-2:~# sudo ss -tlnp | grep 8082
root@vps-2:~#