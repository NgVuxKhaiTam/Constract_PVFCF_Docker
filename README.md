# PVCFC Contract Analysis System

Hệ thống phân tích hợp đồng tự động sử dụng AI cho PVCFC, hỗ trợ chạy trên Docker với GUI.

## 📋 Mô tả

```

## 🚀 Workflow thường dùng

### Lần đầu setup:
```powershell
# 1. Khởi tạo dữ liệu
.\init_data.ps1

# 2. Build image
docker build -t kaggle-automation .

# 3. Chạy container
.\run_docker.bat

# 4. Cấu hình main.py
code main.py  # Sửa thông tin Kaggle

# 5. Chạy automation
docker exec -it kaggle-automation python3 main.py
```

### Sử dụng hàng ngày:
```powershell
# Chạy automation
docker exec -it kaggle-automation python3 main.py

# Xem logs
docker logs kaggle-automation

# Kiểm tra file results
docker exec -it kaggle-automation ls -la /app/files/results/
```

### Khi cần thay đổi cấu hình:
```powershell
# 1. Sửa main.py
code main.py

# 2. Restart container
docker restart kaggle-automation

# 3. Chạy lại
docker exec -it kaggle-automation python3 main.py
```

## 🗑️ Dọn dẹp Docker (Tùy chọn)tích hợp đồng tự động, sử dụng các mô hình AI để trích xuất và phân tích thông tin từ các file PDF hợp đồng. Hệ thống bao gồm:

- **Server AI**: Xử lý PDF và phân tích nội dung bằng VLM (Vision Language Model) và LLM
- **Client Web**: Giao diện web để upload và xử lý file
- **Automation**: Tự động chạy notebook Kaggle và xử lý file theo lịch trình
- **Email Integration**: Tự động lấy URL từ email và xử lý
- **Docker Support**: Containerized environment với GUI support

## 🏗️ Cấu trúc dự án

```
.
├── Dockerfile                        # Docker configuration
├── docker-compose.yml               # Docker Compose setup
├── supervisord.conf                  # Supervisor configuration
├── start.sh                         # Container startup script
├── run_docker.bat                   # Windows Docker runner
├── run_docker.sh                    # Linux/Mac Docker runner
├── init_data.ps1                    # Windows data initialization
├── init_data.sh                     # Linux/Mac data initialization
├── main.py                          # Script chính để chạy automation
├── get_url_via_email.py            # Lấy URL từ email
├── run_kaggle_notebook.py          # Chạy Kaggle notebook (local)
├── run_kaggle_notebook_docker.py   # Chạy Kaggle notebook (Docker)
├── requirements.txt                 # Dependencies
├── processed_files.csv              # File theo dõi đã xử lý
└── files/                          # Thư mục chứa PDF (mounted volume)
    ├── *.pdf                       # PDF files
    └── results/                    # Kết quả JSON
```

## 🚀 Cài đặt và Sử dụng

### Chạy với Docker

#### Yêu cầu:
- Docker Desktop
- 4GB RAM trở lên
- 2GB dung lượng ổ cứng

#### Bước 1: Khởi tạo dữ liệu
```powershell
# Windows
.\init_data.ps1

# Linux/Mac
chmod +x init_data.sh
./init_data.sh
```

#### Bước 2: Build Docker image
```bash
docker build -t kaggle-automation .
```

#### Bước 3: Chạy container
```powershell
# Windows - Cách đơn giản
.\run_docker.bat

# Hoặc chạy manual
docker run -d `
  --name kaggle-automation `
  -p 5900:5900 `
  -p 6080:6080 `
  -v "${PWD}/files:/app/files" `
  -v "${PWD}/processed_files.csv:/app/processed_files.csv" `
  kaggle-automation
```

#### Bước 4: Truy cập GUI
- **Web Browser**: http://localhost:6080
- **VNC Client**: localhost:5900

#### Bước 5: Chạy automation

##### Cách 1: Chạy main.py (Khuyến nghị)
```bash
# Vào container
docker exec -it kaggle-automation bash

# Chạy main.py
python3 main.py
```

##### Cách 2: Chạy notebook riêng lẻ
```bash
# Vào container
docker exec -it kaggle-automation bash

# Chạy script notebook
python3 run_kaggle_notebook_docker.py -u "your_email@gmail.com" -p "your_password" -n "username/notebook-name" -g T4
```

#### Bước 6: Cấu hình main.py

##### Sửa file main.py từ bên ngoài container:
```powershell
# Mở file main.py với editor của bạn (VS Code, Notepad++, v.v.)
code main.py

# Hoặc
notepad main.py
```

##### Sửa file main.py từ bên trong container:
```bash
# Vào container
docker exec -it kaggle-automation bash

# Sửa file với nano
nano main.py

# Hoặc với vi
vi main.py
```

##### Cấu hình cần thiết trong main.py:
```python
# Thông tin tài khoản Kaggle
KAGGLE_ACC_1 = {
    "user": "your_email@gmail.com",           # Email Kaggle của bạn
    "password": "your_password",               # Mật khẩu Kaggle
    "notebook": "username/notebook-name"       # Đường dẫn notebook trên Kaggle
}

# API Key (nếu có)
API_KEY = "your_api_key_here"

# Lịch trình chạy
SCHEDULE_HOUR = 19                    # 7 giờ tối
RUN_MODE = "immediately"              # "immediately", "daily", hoặc "hourly"
```

##### Sau khi sửa xong, restart container:
```bash
# Thoát khỏi container (nếu đang ở trong)
exit

# Restart container để áp dụng thay đổi
docker restart kaggle-automation
```

## ⚙️ Cấu hình chi tiết

### 1. Cấu hình Kaggle (main.py)
```python
KAGGLE_ACC_1 = {
    "user": "your_email@gmail.com",
    "password": "your_password",
    "notebook": "username/notebook-name"
}

API_KEY = "your_api_key_here"
SCHEDULE_HOUR = 19  # 7 giờ tối
RUN_MODE = "immediately"  # hoặc "daily", "hourly"
```

### 2. Cấu hình Email (get_url_via_email.py)
```python
USERNAME = "your_email@gmail.com"
PASSWORD = "your_app_password"  # App password, không phải password thường
```

### 3. Cấu hình Server (utils/client.py)
```python
SERVER_URL = "https://your-ngrok-url.ngrok-free.app"
```

## �️ Dọn dẹp Docker (Tùy chọn)

Nếu bạn muốn bắt đầu từ đầu hoặc gặp lỗi, có thể dọn dẹp tất cả Docker:

### ⚠️ Cảnh báo: Lệnh này sẽ xóa TẤT CẢ containers, images, networks và volumes!

```powershell
# Dừng tất cả containers đang chạy
docker stop $(docker ps -aq)

# Xóa tất cả containers
docker rm $(docker ps -aq)

# Xóa tất cả images
docker rmi $(docker images -q)

# Xóa tất cả volumes
docker volume prune -f

# Xóa tất cả networks
docker network prune -f

# Dọn dẹp hoàn toàn hệ thống
docker system prune -a --volumes -f
```

### Dọn dẹp chỉ project này:
```bash
# Dừng và xóa container kaggle-automation
docker stop kaggle-automation
docker rm kaggle-automation

# Xóa image kaggle-automation
docker rmi kaggle-automation

# Với Docker Compose
docker-compose down --rmi all --volumes
```

## 🔧 Sử dụng nâng cao

### 1. Chạy với Docker Compose
```bash
# Khởi tạo dữ liệu
.\init_data.ps1

# Chạy với Docker Compose
docker-compose up -d

# Xem logs
docker-compose logs -f

# Dừng
docker-compose down
```

### 2. Monitoring và Debug
```bash
# Xem logs container
docker logs kaggle-automation

# Vào container
docker exec -it kaggle-automation bash

# Kiểm tra processes
docker exec -it kaggle-automation ps aux

# Kiểm tra file system
docker exec -it kaggle-automation ls -la /app/files/
```

### 3. Chạy automation với tham số
```bash
# Chạy với GPU T4
docker exec -it kaggle-automation python3 run_kaggle_notebook_docker.py -u "email@gmail.com" -p "password" -n "user/notebook" -g T4

# Chạy với GPU L4
docker exec -it kaggle-automation python3 run_kaggle_notebook_docker.py -u "email@gmail.com" -p "password" -n "user/notebook" -g L4
```

## 📊 Tính năng chính

1. **Tự động xử lý PDF**: Upload và phân tích file PDF hợp đồng
2. **AI Analysis**: Sử dụng VLM và LLM để trích xuất thông tin
3. **Email Integration**: Tự động lấy URL từ email
4. **Kaggle Automation**: Tự động chạy notebook trên Kaggle với GUI
5. **Scheduling**: Chạy theo lịch trình định sẵn
6. **Web Interface**: Giao diện web đơn giản để upload file
7. **Docker Support**: Chạy trong container với VNC/noVNC

## 🛠️ Troubleshooting

### Docker Issues:
```bash
# Container không start được
docker logs kaggle-automation

# Port bị chiếm dụng
netstat -an | findstr :5900
netstat -an | findstr :6080

# Xóa container và chạy lại
docker stop kaggle-automation
docker rm kaggle-automation
docker run -d --name kaggle-automation -p 5900:5900 -p 6080:6080 -v "${PWD}/files:/app/files" kaggle-automation
```

### Local Issues:
```bash
# Kiểm tra Python version
python --version

# Kiểm tra packages
pip list

# Test import
python -c "import requests, selenium, pyautogui, pytz; print('All imports successful')"

# Kiểm tra Firefox
python -c "from selenium import webdriver; from selenium.webdriver.firefox.options import Options; print('Firefox setup OK')"
```

### Common Errors:
1. **ModuleNotFoundError**: Kích hoạt virtual environment và cài lại requirements
2. **Selenium errors**: Kiểm tra Firefox path và geckodriver
3. **Firefox not found**: Cập nhật `binary_location` trong code
4. **Email authentication**: Sử dụng App Password
5. **Permission denied**: Chạy PowerShell as Administrator

## 📝 Chế độ chạy

- **immediately**: Chạy ngay lập tức và lặp lại mỗi tiếng
- **daily**: Chạy hàng ngày vào 7h tối
- **hourly**: Chạy mỗi tiếng

## 📂 Quản lý dữ liệu

- **Dữ liệu persistent**: Tất cả dữ liệu được lưu bên ngoài container
- **Backup**: Chỉ cần backup folder `files/` và `processed_files.csv`
- **Sharing**: Có thể chia sẻ dữ liệu giữa nhiều container
- **Update**: Khi update Docker image, dữ liệu không bị mất

## 🚪 Dừng và dọn dẹp

```bash
# Dừng container
docker stop kaggle-automation
docker rm kaggle-automation

# Dọn dẹp hoàn toàn (cẩn thận!)
docker system prune -a

# Với Docker Compose
docker-compose down
```

## 📞 Liên hệ

Nếu gặp vấn đề, vui lòng liên hệ team phát triển hoặc tạo issue trong repository.

---

*Lưu ý: Đảm bảo cấu hình đúng các thông tin nhạy cảm như API keys, passwords trước khi chạy.*
