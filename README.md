# Bài 4: Quản lý tiến trình nền với nohup và tín hiệu Kill

## 1. Tạo shell script

Tạo file `loop-monitor.sh`:

```bash
#!/bin/bash

while true; do
    echo "System time: $(date)" >> /tmp/monitor.log
    sleep 5
done
```

Cấp quyền thực thi:

```bash
chmod +x loop-monitor.sh
```

Kết quả:

```text
-rwxr-xr-x 1 root root 97 Oct 6 14:35 loop-monitor.sh
```

## 2. Khởi chạy tiến trình nền bằng nohup

Lệnh:

```bash
nohup ./loop-monitor.sh > /dev/null 2>&1 &
```

Kết quả:

```text
[1] 3754
```

PID của tiến trình là `3754`.

## 3. Tìm và kiểm tra PID

Lệnh:

```bash
pgrep -af loop-monitor.sh
```

Kết quả:

```text
3754 /bin/bash ./loop-monitor.sh
```

Kiểm tra bằng `ps`:

```bash
ps aux | grep loop-monitor.sh
```

Kết quả:

```text
root        3754  0.0  0.0   4752  3328 pts/0    S    14:37   0:00 /bin/bash ./loop-monitor.sh
root        3777  0.0  0.0   4096  2048 pts/0    S+   14:38   0:00 grep --color=auto loop-monitor.sh
```

Dòng có PID `3754` là tiến trình `loop-monitor.sh`. Dòng `grep` không phải tiến trình của script.

## 4. Kiểm tra file log

Lệnh:

```bash
tail -n 10 /tmp/monitor.log
```

Kết quả:

```text
System time: Tue Oct  6 14:38:39 UTC 2026
System time: Tue Oct  6 14:38:44 UTC 2026
System time: Tue Oct  6 14:38:49 UTC 2026
System time: Tue Oct  6 14:38:54 UTC 2026
System time: Tue Oct  6 14:38:59 UTC 2026
System time: Tue Oct  6 14:39:04 UTC 2026
System time: Tue Oct  6 14:39:09 UTC 2026
System time: Tue Oct  6 14:39:14 UTC 2026
System time: Tue Oct  6 14:39:19 UTC 2026
System time: Tue Oct  6 14:39:24 UTC 2026
```

Các dòng log được ghi cách nhau 5 giây, chứng minh script đang chạy liên tục.

## 5. Kiểm tra log bằng tail -f

Lệnh:

```bash
tail -f /tmp/monitor.log
```

Kết quả cho thấy log tiếp tục được ghi:

```text
System time: Tue Oct  6 14:39:29 UTC 2026
System time: Tue Oct  6 14:39:34 UTC 2026
System time: Tue Oct  6 14:39:39 UTC 2026
System time: Tue Oct  6 14:39:44 UTC 2026
System time: Tue Oct  6 14:39:49 UTC 2026
System time: Tue Oct  6 14:39:54 UTC 2026
System time: Tue Oct  6 14:39:59 UTC 2026
System time: Tue Oct  6 14:40:04 UTC 2026
```

Nhấn `Ctrl + C` chỉ dừng lệnh `tail -f`, không dừng `loop-monitor.sh`.

## 6. Kiểm tra sau khi ngắt kết nối SSH

Sau khi thoát SSH và đăng nhập lại, kiểm tra:

```bash
pgrep -af loop-monitor.sh
```

Kết quả:

```text
3754 /bin/bash ./loop-monitor.sh
```

Tiến trình vẫn tồn tại với PID `3754`.

Kiểm tra log:

```bash
tail -n 10 /tmp/monitor.log
```

Kết quả:

```text
System time: Tue Oct  6 14:44:30 UTC 2026
System time: Tue Oct  6 14:44:35 UTC 2026
System time: Tue Oct  6 14:44:40 UTC 2026
System time: Tue Oct  6 14:44:45 UTC 2026
System time: Tue Oct  6 14:44:50 UTC 2026
System time: Tue Oct  6 14:44:55 UTC 2026
System time: Tue Oct  6 14:45:00 UTC 2026
System time: Tue Oct  6 14:45:05 UTC 2026
System time: Tue Oct  6 14:45:10 UTC 2026
System time: Tue Oct  6 14:45:15 UTC 2026
```

Sau đó kiểm tra lại:

```bash
pgrep -af loop-monitor.sh
```

Kết quả:

```text
3754 /bin/bash ./loop-monitor.sh
```

Kiểm tra bằng `ps`:

```bash
ps aux | grep loop-monitor.sh
```

Kết quả:

```text
root        3754  0.0  0.0   4752  3328 pts/0    S    14:37   0:00 /bin/bash ./loop-monitor.sh
root        4095  0.0  0.0   4096  2048 pts/1    S+   14:48   0:00 grep --color=auto loop-monitor.sh
```

Kết quả chứng minh tiến trình vẫn hoạt động sau khi phiên SSH cũ bị ngắt.

## 7. Dừng tiến trình bằng SIGTERM

Dùng PID `3754`:

```bash
kill -15 3754
```

Kiểm tra:

```bash
pgrep -af loop-monitor.sh
```

Kết quả:

```text

```

Không còn tiến trình `loop-monitor.sh`.

Kiểm tra bằng:

```bash
ps aux | grep loop-monitor.sh
```

Kết quả chỉ còn:

```text
root        4106  0.0  0.0   4096  2048 pts/1    S+   14:49   0:00 grep --color=auto loop-monitor.sh
```

Tiến trình đã được dừng thành công bằng `SIGTERM (15)`.

Không cần sử dụng `SIGKILL (9)` vì `SIGTERM` đã dừng tiến trình thành công.
