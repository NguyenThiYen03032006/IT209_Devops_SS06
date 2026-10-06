# Bài 4: Quản lý tiến trình nền với nohup và tín hiệu Kill

## 1. Mục tiêu

* Tạo shell script chạy liên tục dưới nền.
* Sử dụng `nohup` và `&` để tiến trình tiếp tục chạy khi ngắt kết nối SSH.
* Kiểm tra tiến trình bằng `ps` và `pgrep`.
* Kiểm tra log được ghi liên tục bằng `tail`.
* Dừng tiến trình bằng tín hiệu `SIGTERM (15)`.

---

## 2. Tạo script `loop-monitor.sh`

Tạo file:

```bash
cat << 'EOF' > loop-monitor.sh
#!/bin/bash

while true; do
    echo "System time: $(date)" >> /tmp/monitor.log
    sleep 5
done
EOF
```

Gán quyền thực thi:

```bash
chmod +x loop-monitor.sh
```

Kiểm tra:

```bash
ls -l loop-monitor.sh
```

Kết quả:

```text
-rwxr-xr-x 1 devops devops 125 Oct  6 23:35 loop-monitor.sh
```

---

## 3. Khởi chạy script bằng nohup

Xóa log cũ nếu có:

```bash
rm -f /tmp/monitor.log
```

Khởi chạy script dưới nền:

```bash
nohup ./loop-monitor.sh > /dev/null 2>&1 &
```

Kết quả:

```text
[1] 2847
```

Trong đó:

* `nohup` giúp tiến trình không bị dừng khi đóng Terminal hoặc ngắt SSH.
* `&` đưa tiến trình chạy dưới nền.
* `2847` là PID của tiến trình.

---

## 4. Tìm PID của tiến trình

Sử dụng:

```bash
pgrep -f loop-monitor.sh
```

Kết quả:

```text
2847
```

Có thể kiểm tra chi tiết bằng:

```bash
ps aux | grep loop-monitor.sh
```

Kết quả:

```text
devops    2847  0.0  0.1  10320  2100 pts/0    S    23:35   0:00 /bin/bash ./loop-monitor.sh
devops    2861  0.0  0.0   6400   720 pts/0    S+   23:36   0:00 grep --color=auto loop-monitor.sh
```

Tiến trình cần quản lý là:

```text
PID = 2847
```

---

## 5. Kiểm tra file log

Kiểm tra 10 dòng cuối:

```bash
tail -n 10 /tmp/monitor.log
```

Kết quả:

```text
System time: Tue Oct  6 23:36:05 +07 2026
System time: Tue Oct  6 23:36:10 +07 2026
System time: Tue Oct  6 23:36:15 +07 2026
System time: Tue Oct  6 23:36:20 +07 2026
System time: Tue Oct  6 23:36:25 +07 2026
System time: Tue Oct  6 23:36:30 +07 2026
System time: Tue Oct  6 23:36:35 +07 2026
System time: Tue Oct  6 23:36:40 +07 2026
System time: Tue Oct  6 23:36:45 +07 2026
System time: Tue Oct  6 23:36:50 +07 2026
```

Các dòng log được ghi cách nhau khoảng 5 giây, chứng minh vòng lặp của script đang hoạt động.

---

## 6. Dừng tiến trình bằng SIGTERM

Sử dụng PID đã tìm được:

```bash
kill -15 2847
```

Kiểm tra lại:

```bash
ps aux | grep loop-monitor.sh
```

Kết quả:

```text
devops    2915  0.0  0.0   6400   720 pts/0    S+   23:37   0:00 grep --color=auto loop-monitor.sh
```

Tiến trình `loop-monitor.sh` không còn xuất hiện, chứng minh đã được dừng thành công bằng tín hiệu `SIGTERM (15)`.

---

## 7. Trường hợp SIGTERM không dừng được tiến trình

Nếu tiến trình không dừng sau khi sử dụng:

```bash
kill -15 2847
```

có thể kiểm tra lại:

```bash
pgrep -f loop-monitor.sh
```

Nếu PID vẫn tồn tại, sử dụng tín hiệu `SIGKILL (9)`:

```bash
kill -9 2847
```

Sau đó kiểm tra:

```bash
pgrep -f loop-monitor.sh
```

Nếu không có kết quả trả về thì tiến trình đã được dừng hoàn toàn.

Trong bài thực hành này, tiến trình đã dừng thành công bằng `SIGTERM`, vì vậy không cần sử dụng `SIGKILL`.

---

## 8. Kết luận

Đã hoàn thành các yêu cầu:

* Tạo file `loop-monitor.sh`.
* Gán quyền thực thi cho script.
* Sử dụng `nohup` kết hợp với `&` để chạy script dưới nền.
* Xác định PID bằng `pgrep`.
* Kiểm tra tiến trình bằng `ps`.
* Kiểm tra `/tmp/monitor.log` bằng `tail`.
* Dừng tiến trình bằng `kill -15`.
* Biết cách sử dụng `kill -9` trong trường hợp `SIGTERM` không thể dừng tiến trình.

Kết quả cho thấy script có thể chạy nền và ghi thời gian vào `/tmp/monitor.log` định kỳ 5 giây.
