# Hướng Dẫn Kiểm Thử Hiệu Năng với JMeter

## 1. Chuẩn Bị

### 1.1 Cài Đặt JMeter

**Windows:**
```bash
# Download từ: https://jmeter.apache.org/download_jmeter.cgi
# Hoặc dùng Chocolatey/WinGet
choco install jmeter
# hoặc
winget install Apache.JMeter
```

**Kiểm tra cài đặt:**
```bash
jmeter --version
```

### 1.2 Khởi Động Backend

```bash
# Kích hoạt virtual environment
.venv\Scripts\Activate.ps1

# Cài đặt dependencies (nếu chưa có)
pip install -r backend/requirements.txt

# Khởi động server (mặc định port 8000)
cd backend
python -m uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

**Kiểm tra:** http://localhost:8000/docs (Swagger UI)

---

## 2. Các Endpoints Chính Để Test

Dựa trên cấu trúc API, những endpoint quan trọng:

| Endpoint | Method | Mô Tả | Độ Ưu Tiên |
|----------|--------|-------|-----------|
| `/api/v1/healthz` | GET | Kiểm tra sức khỏe server | ⭐⭐⭐⭐⭐ |
| `/api/v1/auth/register` | POST | Đăng ký tài khoản | ⭐⭐⭐⭐ |
| `/api/v1/auth/login` | POST | Đăng nhập | ⭐⭐⭐⭐⭐ |
| `/api/v1/detect` | POST | Phát hiện bệnh cây (multipart) | ⭐⭐⭐⭐⭐ |
| `/api/v1/users/me` | GET | Lấy thông tin người dùng | ⭐⭐⭐⭐ |
| `/api/v1/devices` | GET | Lấy danh sách thiết bị | ⭐⭐⭐⭐ |
| `/api/v1/sensors` | GET | Lấy dữ liệu cảm biến | ⭐⭐⭐⭐ |
| `/api/v1/dashboard` | GET | Lấy dữ liệu dashboard | ⭐⭐⭐ |

---

## 3. Tạo Test Plan JMeter

### 3.1 Mở JMeter GUI

```bash
jmeter
```

### 3.2 Tạo Test Plan Cơ Bản

**Bước 1:** Thêm Thread Group
- Right-click `Test Plan` → Add → Threads (Users) → Thread Group
- Cấu hình:
  - **Number of Threads (users):** 10 (bắt đầu nhỏ)
  - **Ramp-Up Period (seconds):** 10 (tăng dần trong 10 giây)
  - **Loop Count:** 5 (mỗi thread chạy 5 lần)

**Bước 2:** Thêm HTTP Request Defaults
- Right-click `Thread Group` → Add → Config Element → HTTP Request Defaults
- Cấu hình:
  - **Server Name or IP:** `localhost`
  - **Port Number:** `8000`
  - **Protocol:** `http`

**Bước 3:** Thêm HTTP Request (Health Check)
- Right-click `Thread Group` → Add → Sampler → HTTP Request
- Cấu hình:
  - **Path:** `/api/v1/healthz`
  - **Method:** `GET`

**Bước 4:** Thêm Listeners
- Right-click `Thread Group` → Add → Listener → Summary Report
- Right-click `Thread Group` → Add → Listener → View Results Tree
- Right-click `Thread Group` → Add → Listener → Response Time Graph

### 3.3 Chạy Test Đơn Giản

1. Click **Run** → **Start** (hoặc Ctrl+Enter)
2. Xem kết quả trong Summary Report

---

## 4. Test Plan Nâng Cao (Multipart Image Upload)

### 4.1 Tạo Test Plan Cho `/detect`

Vì endpoint `/detect` yêu cầu file image, ta cần:

**Bước 1:** Chuẩn bị ảnh test
```bash
# Copy một file ảnh vào thư mục test
# VD: backend/test_image.jpg
```

**Bước 2:** Thêm HTTP Request cho Detection
- Path: `/api/v1/detect`
- Method: `POST`
- **Body Data:**
  - Tab "Body Data" → thêm parameters:
    - Name: `file` | Value: `path/to/test_image.jpg` | File upload: ✓

**Bước 3:** Thêm Authentication (nếu cần)
- Nếu endpoint yêu cầu token JWT:
  - Right-click HTTP Request → Add → Config Element → HTTP Header Manager
  - Thêm header:
    ```
    Name: Authorization
    Value: Bearer <YOUR_JWT_TOKEN>
    ```

---

## 5. Test Plan Với Authentication

### 5.1 Sử dụng Post-Processor

**Kịch bản:** Login → Lấy token → Dùng token cho requests khác

**Bước 1:** Thêm HTTP Request cho Login
- Path: `/api/v1/auth/login`
- Method: `POST`
- Body:
```json
{
  "email": "test@example.com",
  "password": "password123"
}
```

**Bước 2:** Thêm JSON Extractor
- Right-click HTTP Request (Login) → Add → Post Processor → JSON Extractor
- Cấu hình:
  - **Names of created variables:** `token`
  - **JSON Path Expressions:** `$.access_token`

**Bước 3:** Dùng Token Ở Requests Tiếp Theo
- Thêm HTTP Header Manager trong các request khác
- Header: `Authorization: Bearer ${token}`

---

## 6. Chạy Test Và Phân Tích

### 6.1 Chạy Ở Dòng Lệnh (CLI)

```bash
# Chạy test plan
jmeter -n -t my_test_plan.jmx -l results.jtl -j jmeter.log

# Tạo báo cáo HTML
jmeter -g results.jtl -o report/
```

### 6.2 Giải Thích Các Chỉ Số

| Chỉ Số | Ý Nghĩa |
|--------|---------|
| **Avg** | Thời gian phản hồi trung bình (ms) |
| **Min** | Thời gian phản hồi nhanh nhất (ms) |
| **Max** | Thời gian phản hồi chậm nhất (ms) |
| **Std. Dev.** | Độ lệch chuẩn (đo độ biến động) |
| **Error %** | Tỷ lệ lỗi (%) |
| **Throughput** | Số request/giây |

### 6.3 Mục Tiêu Hiệu Năng

Cho ứng dụng loại này:
- ✅ **Health check:** < 50ms, 0% error
- ✅ **Login:** < 500ms, 0% error
- ✅ **Detection (upload):** < 3000ms, < 5% error
- ✅ **GET endpoints:** < 200ms, 0% error
- ✅ **Throughput:** > 100 req/s (tùy khả năng server)

---

## 7. Tệp Test Plan Mẫu

Lưu file dưới: `jmeter_test_plans/basic_test.jmx`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jmeterTestPlan version="1.2" properties="5.0" jmeter="5.6">
  <hashTree>
    <TestPlan guiclass="TestPlanGui" testclass="TestPlan" testname="Plant Health API Load Test" enabled="true">
      <elementProp name="TestPlan.user_defined_variables" elementType="Arguments" guiclass="ArgumentsPanel" testclass="Arguments" testname="User Defined Variables" enabled="true">
        <collectionProp name="Arguments.arguments"/>
      </elementProp>
      <stringProp name="TestPlan.user_define_classpath"></stringProp>
      <boolProp name="TestPlan.functional_mode">false</boolProp>
      <boolProp name="TestPlan.serialize_threadgroups">false</boolProp>
      <elementProp name="TestPlan.test_fragments" elementType="CollectionProperty" guiclass="CollectionProperty" testclass="CollectionProperty" testname="Test Plan" enabled="true">
        <collectionProp name="elementProp"/>
      </elementProp>
      <stringProp name="TestPlan.comments"></stringProp>
    </TestPlan>
    <hashTree>
      <ThreadGroup guiclass="ThreadGroupGui" testclass="ThreadGroup" testname="Users" enabled="true">
        <elementProp name="ThreadGroup.main_controller" elementType="LoopController" guiclass="LoopControlPanel" testclass="LoopController" testname="Loop Controller" enabled="true">
          <boolProp name="LoopController.continue_forever">false</boolProp>
          <stringProp name="LoopController.loops">5</stringProp>
        </elementProp>
        <stringProp name="ThreadGroup.num_threads">10</stringProp>
        <stringProp name="ThreadGroup.ramp_time">10</stringProp>
        <longProp name="ThreadGroup.start_time">1513856572000</longProp>
        <longProp name="ThreadGroup.end_time">1513856572000</longProp>
        <boolProp name="ThreadGroup.scheduler">false</boolProp>
        <stringProp name="ThreadGroup.duration"></stringProp>
        <stringProp name="ThreadGroup.delay"></stringProp>
        <boolProp name="ThreadGroup.same_user_on_next_iteration">true</boolProp>
      </ThreadGroup>
      <hashTree>
        <ConfigTestElement guiclass="HttpDefaultsGui" testclass="ConfigTestElement" testname="HTTP Request Defaults" enabled="true">
          <elementProp name="HTTPsampler.Arguments" elementType="Arguments" guiclass="HTTPArgumentsPanel" testclass="Arguments" testname="User Defined Variables" enabled="true">
            <collectionProp name="Arguments.arguments"/>
          </elementProp>
          <stringProp name="HTTPSampler.domain">localhost</stringProp>
          <stringProp name="HTTPSampler.port">8000</stringProp>
          <stringProp name="HTTPSampler.protocol">http</stringProp>
          <stringProp name="HTTPSampler.contentEncoding"></stringProp>
          <stringProp name="HTTPSampler.path"></stringProp>
          <stringProp name="HTTPSampler.concurrentPool">4</stringProp>
          <stringProp name="HTTPSampler.connect_timeout"></stringProp>
          <stringProp name="HTTPSampler.response_timeout"></stringProp>
        </ConfigTestElement>
        <hashTree/>
        <HTTPSampler guiclass="HttpTestSampleGui" testclass="HTTPSampler" testname="GET /healthz" enabled="true">
          <elementProp name="HTTPsampler.Arguments" elementType="Arguments" guiclass="HTTPArgumentsPanel" testclass="Arguments" testname="User Defined Variables" enabled="true">
            <collectionProp name="Arguments.arguments"/>
          </elementProp>
          <stringProp name="HTTPSampler.domain"></stringProp>
          <stringProp name="HTTPSampler.port"></stringProp>
          <stringProp name="HTTPSampler.protocol"></stringProp>
          <stringProp name="HTTPSampler.contentEncoding"></stringProp>
          <stringProp name="HTTPSampler.path">/api/v1/healthz</stringProp>
          <stringProp name="HTTPSampler.method">GET</stringProp>
          <boolProp name="HTTPSampler.follow_redirects">true</boolProp>
          <boolProp name="HTTPSampler.auto_redirects">false</boolProp>
          <boolProp name="HTTPSampler.use_keepalive">true</boolProp>
          <boolProp name="HTTPSampler.DO_MULTIPART_POST">false</boolProp>
          <stringProp name="HTTPSampler.embedded_url_re"></stringProp>
          <stringProp name="HTTPSampler.connect_timeout"></stringProp>
          <stringProp name="HTTPSampler.response_timeout"></stringProp>
        </HTTPSampler>
        <hashTree/>
        <ResultCollector guiclass="SummaryReport" testclass="ResultCollector" testname="Summary Report" enabled="true">
          <objProp>
            <name>saveConfig</name>
            <value class="SampleSaveConfiguration">
              <time>true</time>
              <latency>true</latency>
              <timestamp>true</timestamp>
              <success>true</success>
              <label>true</label>
              <code>true</code>
              <message>true</message>
              <threadName>true</threadName>
              <dataType>true</dataType>
              <encoding>false</encoding>
              <assertions>true</assertions>
              <subresults>true</subresults>
              <responseData>false</responseData>
              <samplerData>false</samplerData>
              <xml>false</xml>
              <fieldNames>true</fieldNames>
              <responseHeaders>false</responseHeaders>
              <requestHeaders>false</requestHeaders>
              <responseDataOnError>false</responseDataOnError>
              <saveAssertionResultsFailureMessage>true</saveAssertionResultsFailureMessage>
              <assertionsResultsToSave>0</assertionsResultsToSave>
              <bytes>true</bytes>
              <sentBytes>true</sentBytes>
              <url>true</url>
              <fileName>true</fileName>
              <hostname>true</hostname>
              <threadCounts>true</threadCounts>
              <sampleCount>true</sampleCount>
              <idleTime>true</idleTime>
              <connectTime>true</connectTime>
            </value>
          </objProp>
          <stringProp name="filename"></stringProp>
        </ResultCollector>
        <hashTree/>
      </hashTree>
    </hashTree>
  </hashTree>
</jmeterTestPlan>
```

---

## 8. Kiểm Thử Trên Production-Like Environment

### 8.1 Cấu Hình Docker Compose

```yaml
version: '3.8'
services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - DB_HOST=db
      - DB_USER=root
      - DB_PASS=password
    depends_on:
      - db
  
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: ai_plant_db
    volumes:
      - ./db:/docker-entrypoint-initdb.d
    ports:
      - "3306:3306"
```

Chạy:
```bash
docker-compose -f infra/docker/docker-compose.dev.yml up
```

### 8.2 Chạy Test Sau Khi Deploy

```bash
jmeter -n -t jmeter_test_plans/basic_test.jmx \
        -l results_production.jtl \
        -j jmeter_production.log \
        -Jhost=localhost \
        -Jport=8000
```

---

## 9. Mẹo Tối Ưu Hóa

### 9.1 Tối Ưu Backend Trước Khi Test
```bash
# Kiểm tra logs
tail -f backend/app.log

# Theo dõi CPU/Memory
# Windows Task Manager hoặc
# Get-Process python | Select-Object Name, WorkingSet
```

### 9.2 Tối Ưu JMeter
- Tắt GUI khi chạy test lớn → dùng CLI (`-n` flag)
- Tăng heap size nếu test quá lớn:
```bash
set JVM_ARGS=-Xmx2048m
jmeter -n -t test.jmx -l results.jtl
```

### 9.3 Cách Debug Các Lỗi
- Bật "View Results Tree" → xem chi tiết request/response
- Thêm **Assertions** để kiểm tra response code
- Thêm **Response Assertion** để kiểm tra content

---

## 10. Ví Dụ Assertions

Right-click HTTP Sampler → Add → Assertions → Response Assertion

```
Pattern to test: ${response_code}
Test type: Equals
Pattern: 200
```

---

## Tài Liệu Tham Khảo

- [JMeter Official Docs](https://jmeter.apache.org/usermanual/)
- [JMeter Best Practices](https://jmeter.apache.org/usermanual/best-practices.html)
- [API Testing with JMeter](https://jmeter.apache.org/usermanual/component_reference.html)

---

**Bắt đầu nào! 🚀**
