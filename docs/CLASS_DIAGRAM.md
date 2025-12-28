# Biểu Đồ Lớp Hệ Thống (Class Diagram)

## 📋 Tổng Quan

Hệ thống quản lý phát hiện bệnh cây trồng với AI bao gồm các phần chính:
- **Quản lý Người Dùng** (Users & Authentication)
- **Quản lý Thiết Bị** (Devices & IoT)
- **Cảm Biến & Dữ Liệu** (Sensor Readings)
- **Phát Hiện Bệnh** (Disease Detection & AI)
- **Hệ Thống Hỗ Trợ** (Support & Notifications)
- **Chatbot & Cài Đặt** (Chatbot & User Settings)

---

## 1. Module Người Dùng & Xác Thực

```
┌─────────────────────────────────────────────────────────────────┐
│                      USERS & AUTHENTICATION                      │
└─────────────────────────────────────────────────────────────────┘

┌──────────────────┐         ┌──────────────────┐
│     Users        │         │      Role        │
├──────────────────┤         ├──────────────────┤
│ user_id: BigInt  │◄────────│ role_id: BigInt  │
│ role_id: FK      │         │ role_type: Enum  │
│ username: Str    │         │ description: Str │
│ email: Str       │         └──────────────────┘
│ phone: Str       │                  ▲
│ avt_url: Str     │                  │ 1
│ address: Str     │                  │
│ status: Enum     │                  │ (admin, support,
│ password: Str    │                  │  viewer, support_admin)
│ failed_login: Int│
│ locked: DateTime │
│ created_at: DT   │
│ updated_at: DT   │
└──────────────────┘
      ▲     ▲
      │     │
      │     └──────────────┬──────────────────┐
      │                    │                  │
      │          ┌─────────────────┐  ┌───────────────────┐
      │          │   AuthAccount   │  │  UserSettings     │
      │          ├─────────────────┤  ├───────────────────┤
      │          │ auth_id: BigInt │  │ user_setting_id   │
      │          │ user_id: FK     │  │ user_id: FK       │
      │          │ provider: Enum  │  │ color: Str        │
      │          │ provider_user   │  │ font_size: Str    │
      │          │ phone_verified  │  │ language: Str     │
      │          │ created_at: DT  │  │ notification_en   │
      │          └─────────────────┘  │ auto_connect: Bool│
      │                                │ share_data: Bool  │
      │                                └───────────────────┘
      │
      ├─────────────────────────────────────────┐
      │                                         │
      ▼                                         ▼
┌─────────────────┐                  ┌──────────────────┐
│  Notifications  │                  │ SupportTicket    │
├─────────────────┤                  ├──────────────────┤
│ notif_id: BigInt│                  │ ticket_id: BigInt│
│ user_id: FK     │                  │ user_id: FK      │
│ sender_id: FK   │                  │ title: Str       │
│ title: Str      │                  │ description: Text│
│ description     │                  │ status: Enum     │
│ created_at: DT  │                  │ created_at: DT   │
│ read_at: DT     │                  │                  │
│ resend_of: FK   │                  └──────────────────┘
└─────────────────┘                         │
                                            │ 1..*
                                            ▼
                                  ┌──────────────────────┐
                                  │  SupportMessage      │
                                  ├──────────────────────┤
                                  │ message_id: BigInt   │
                                  │ ticket_id: FK        │
                                  │ sender_id: FK        │
                                  │ message: Text        │
                                  │ attachment_url: Str  │
                                  │ created_at: DT       │
                                  └──────────────────────┘

RoleType Enum:
├─ support
├─ viewer
├─ admin
└─ support_admin

UserStatus Enum:
├─ active
└─ inactive

Provider Enum:
├─ sdt (phone)
├─ gg (Google)
├─ fb (Facebook)
└─ email

TicketStatus Enum:
├─ processing
└─ processed
```

---

## 2. Module Thiết Bị & IoT

```
┌─────────────────────────────────────────────────────────────────┐
│                    DEVICES & IOT MANAGEMENT                      │
└─────────────────────────────────────────────────────────────────┘

┌──────────────────┐         ┌──────────────────┐
│   DeviceType     │         │     Device       │
├──────────────────┤         ├──────────────────┤
│ device_type_id   │◄────────│ device_id: BigInt│
│ device_type_name │         │ user_id: FK      │
│ has_stream: Bool │         │ name: Str        │
│ status: Enum     │         │ device_type_id FK
│ created_at: DT   │         │ parent_device_id │
└──────────────────┘         │ serial_no: Str   │
                             │ location: Str    │
                             │ status: Enum     │
                             │ stream_url: Str  │
                             │ created_at: DT   │
                             │ updated_at: DT   │
                             └──────────────────┘
                                    ▲
                                    │ 1
                                    │
                                    ├─────────────────┬──────────────────┐
                                    │                 │                  │
                        ┌───────────────────┐  ┌────────────────┐  ┌────────────────┐
                        │ SensorReadings    │  │  DeviceLogs    │  │      Img       │
                        ├───────────────────┤  ├────────────────┤  ├────────────────┤
                        │ reading_id: BigInt│  │ log_id: BigInt │  │ img_id: BigInt │
                        │ device_id: FK     │  │ device_id: FK  │  │ device_id: FK  │
                        │ metric: Str       │  │ event_type: En │  │ user_id: FK    │
                        │ value_num: Dec    │  │ description    │  │ file_url: Str  │
                        │ unit: Str         │  │ created_at: DT │  │ source_type    │
                        │ status: Enum      │  └────────────────┘  │ created_at: DT │
                        │ recorded_at: DT   │                       └────────────────┘
                        └───────────────────┘                               │
                                                                            │ 1..*
DeviceStatus Enum:                                                          ▼
├─ active                                                          ┌────────────────┐
├─ maintain                                                        │   Detection    │
└─ inactive                                                        ├────────────────┤
                                                                    │ detection_id   │
DeviceTypeStatus Enum:                                             │ img_id: FK     │
├─ active                                                         │ disease_id: FK │
└─ inactive                                                       │ confidence     │
                                                                    │ description    │
SensorStatus Enum:                                                 │ treatment_     │
├─ ok                                                             │ bbox: JSON     │
├─ error                                                          │ review_status  │
└─ missing                                                        │ model_version  │
                                                                    │ created_at: DT │
DeviceEventType Enum:                                             └────────────────┘
├─ online                                                                 │
├─ offline                                                                │ n..*
├─ error                                                                  │
└─ maintenance                                                            ▼
                                                                   ┌────────────────┐
SourceType Enum:                                                  │    Disease     │
├─ camera                                                         ├────────────────┤
└─ upload                                                         │ disease_id     │
                                                                    │ name: Str      │
ReviewStatus Enum:                                                │ description    │
├─ pending                                                        │ treatment_     │
├─ approved                                                       │ created_at: DT │
└─ rejected                                                       └────────────────┘
```

---

## 3. Module Phát Hiện Bệnh & AI

```
┌─────────────────────────────────────────────────────────────────┐
│            DISEASE DETECTION & AI INFERENCE                      │
└─────────────────────────────────────────────────────────────────┘

┌──────────────────────────┐
│    YoloDetector          │
├──────────────────────────┤
│ model: YOLO              │
│ names: Dict[int, str]    │
├──────────────────────────┤
│ predict_bytes(bytes)     │
│  → Dict[str, Any]        │
│ - Tải model từ ML folder │
│ - Inference ảnh          │
│ - Trả output với bbox    │
└──────────────────────────┘
         ▲
         │ uses
         │
┌──────────────────────────┐
│  DetectService           │
├──────────────────────────┤
│ save_image_to_disk()     │
│ ensure_disease()         │
│ _normalize_bbox()        │
│ process_detection()      │
│ create_detection()       │
├──────────────────────────┤
│ - Lưu ảnh vào media/     │
│ - Tìm hoặc tạo Disease   │
│ - Chuẩn hóa bbox format  │
│ - Lưu kết quả Detection  │
└──────────────────────────┘
         ▲
         │ uses
         │
┌──────────────────────────┐
│  InferenceService        │
├──────────────────────────┤
│ predict_bytes()          │
│ get_disease_for_result() │
│ _no_detection_explain()  │
├──────────────────────────┤
│ - Gọi YoloDetector       │
│ - Map nhãn → Tiếng Việt  │
│ - Tạo giải thích nếu ko  │
│   phát hiện được bệnh    │
└──────────────────────────┘
         ▲
         │ uses
         │
VN_LABELS = {
  "pomelo_leaf_healthy" → "Lá bưởi khỏe mạnh"
  "pomelo_leaf_miner" → "Lá bưởi bị sâu vẽ bùa"
  "pomelo_leaf_yellowing" → "Lá bưởi bị vàng lá"
  "pomelo_fruit_healthy" → "Quả bưởi khỏe mạnh"
  "pomelo_fruit_scorch" → "Quả bưởi bị cháy / nám vỏ"
}
```

---

## 4. Module Chatbot & Tương Tác

```
┌─────────────────────────────────────────────────────────────────┐
│                   CHATBOT & INTERACTION                          │
└─────────────────────────────────────────────────────────────────┘

┌────────────────────┐
│    Chatbot         │
├────────────────────┤
│ chatbot_id: BigInt │
│ user_id: FK        │
│ created_at: DT     │
│ end_at: DT         │
│ status: Enum       │
├────────────────────┤
│ User 1..*          │
└────────────────────┘
         │ 1
         │
         ├─ 1..*
         │
         ▼
┌────────────────────┐
│ ChatbotDetail      │
├────────────────────┤
│ detail_id: BigInt  │
│ chatbot_id: FK     │
│ question: Text     │
│ answer: Text       │
│ created_at: DT     │
└────────────────────┘

┌────────────────────────────┐
│  ChatbotService            │
├────────────────────────────┤
│ start_session()            │
│ add_question_answer()      │
│ end_session()              │
├────────────────────────────┤
│ - Tạo session chatbot      │
│ - Lưu Q&A                  │
│ - Kết thúc session         │
└────────────────────────────┘

ChatbotStatus Enum:
├─ active
└─ ended
```

---

## 5. Module Xác Thực & Bảo Mật

```
┌─────────────────────────────────────────────────────────────────┐
│                 AUTHENTICATION & SECURITY                        │
└─────────────────────────────────────────────────────────────────┘

┌──────────────────────────┐
│   AuthJWTService         │
├──────────────────────────┤
│ make_access_token()      │
│ decode_access_token()    │
│ make_refresh_token()     │
│ decode_refresh_token()   │
├──────────────────────────┤
│ - Tạo JWT token         │
│ - Xác thực token        │
│ - Quản lý thời gian hết  │
│   hạn token             │
└──────────────────────────┘
         ▲
         │ uses
         │
┌──────────────────────────┐
│   IdentityVerify         │
├──────────────────────────┤
│ verify_phone()           │
│ verify_email()           │
│ send_otp()               │
│ verify_otp()             │
├──────────────────────────┤
│ - OTP verification      │
│ - Phone & email check   │
└──────────────────────────┘
         ▲
         │ uses
         │
┌──────────────────────────┐
│   OTPCodes               │
├──────────────────────────┤
│ generate_otp()           │
│ store_otp()              │
│ validate_otp()           │
├──────────────────────────┤
│ - Tạo mã OTP            │
│ - Lưu vào cache/DB      │
│ - Xác thực OTP          │
└──────────────────────────┘

┌──────────────────────────┐
│   PasswordService        │
├──────────────────────────┤
│ hash_password()          │
│ verify_password()        │
│ reset_password()         │
├──────────────────────────┤
│ - Hash mật khẩu         │
│ - Xác thực mật khẩu     │
│ - Reset/change password │
└──────────────────────────┘
```

---

## 6. Module Dịch Vụ & Tiện Ích

```
┌─────────────────────────────────────────────────────────────────┐
│                 SERVICES & UTILITIES                             │
└─────────────────────────────────────────────────────────────────┘

┌────────────────────────┐  ┌────────────────────────┐
│  DeviceService         │  │  DeviceTypeService     │
├────────────────────────┤  ├────────────────────────┤
│ list_devices()         │  │ list_device_types()    │
│ get_device()           │  │ get_device_type()      │
│ list_devices_of_user() │  │ create_device_type()   │
│ create_device()        │  │ update_device_type()   │
│ update_device()        │  │ delete_device_type()   │
│ delete_device()        │  └────────────────────────┘
└────────────────────────┘

┌────────────────────────┐  ┌────────────────────────┐
│  DashboardService      │  │  ReportService         │
├────────────────────────┤  ├────────────────────────┤
│ get_dashboard()        │  │ generate_report()      │
│ get_stats()            │  │ export_report()        │
│ get_latest_detections()│  │ get_report_history()   │
├────────────────────────┤  ├────────────────────────┤
│ - Thống kê tổng quan   │  │ - Tạo báo cáo         │
│ - Dữ liệu dashboard    │  │ - Export CSV/PDF      │
│ - Phát hiện gần đây    │  │ - Lịch sử báo cáo     │
└────────────────────────┘  └────────────────────────┘

┌────────────────────────┐  ┌────────────────────────┐
│  NotificationService   │  │  NewsService           │
├────────────────────────┤  ├────────────────────────┤
│ send_notification()    │  │ fetch_news()           │
│ get_notifications()    │  │ parse_news()           │
│ mark_as_read()         │  │ cache_news()           │
├────────────────────────┤  ├────────────────────────┤
│ - Gửi thông báo       │  │ - Lấy tin tức cây      │
│ - Lấy danh sách        │  │ - Parse và lưu cache   │
│ - Đánh dấu đã xem      │  │ - Phục vụ FE           │
└────────────────────────┘  └────────────────────────┘

┌────────────────────────┐  ┌────────────────────────┐
│  LLMService            │  │  WeatherService        │
├────────────────────────┤  ├────────────────────────┤
│ generate_explanation() │  │ get_weather()          │
│ ask_question()         │  │ get_forecast()         │
│ get_treatment()        │  │ map_weather_data()     │
├────────────────────────┤  ├────────────────────────┤
│ - Sử dụng LLM (OpenAI)│  │ - Gọi OpenWeather API │
│ - Giải thích bệnh      │  │ - Lưu cache           │
│ - Hỏi đáp AI           │  │ - Trả về dữ liệu      │
└────────────────────────┘  └────────────────────────┘

┌────────────────────────┐  ┌────────────────────────┐
│  CameraService         │  │  StreamService         │
├────────────────────────┤  ├────────────────────────┤
│ capture_frame()        │  │ start_stream()         │
│ process_stream()       │  │ stop_stream()          │
├────────────────────────┤  │ get_stream_url()       │
│ - Quản lý camera       │  ├────────────────────────┤
│ - Capture frame        │  │ - Quản lý stream HLS   │
│ - Xử lý video          │  │ - Start/stop stream    │
└────────────────────────┘  └────────────────────────┘

┌────────────────────────────────────────┐
│  AutoDetectionService                  │
├────────────────────────────────────────┤
│ start_auto_detection()                 │
│ stop_auto_detection()                  │
│ schedule_detection()                   │
├────────────────────────────────────────┤
│ - Tự động detect từ camera            │
│ - Lên lịch detect định kỳ             │
│ - Quản lý scheduler                    │
└────────────────────────────────────────┘

┌────────────────────────────────────────┐
│  SchedulerService                      │
├────────────────────────────────────────┤
│ schedule_job()                         │
│ cancel_job()                           │
│ get_scheduled_jobs()                   │
├────────────────────────────────────────┤
│ - Lên lịch tác vụ định kỳ             │
│ - Quản lý APScheduler                  │
│ - Trigger auto-detection               │
└────────────────────────────────────────┘
```

---

## 7. Module Phân Quyền & Thông Báo

```
┌─────────────────────────────────────────────────────────────────┐
│              PERMISSIONS & NOTIFICATIONS                         │
└─────────────────────────────────────────────────────────────────┘

┌────────────────────────────────────────┐
│  PermissionService                     │
├────────────────────────────────────────┤
│ check_permission()                     │
│ has_role()                             │
│ require_permission()                   │
├────────────────────────────────────────┤
│ - Kiểm tra quyền hạn                  │
│ - Xác thực role                       │
│ - Middleware decorator                │
└────────────────────────────────────────┘
         ▲
         │ uses
         │
┌────────────────────────────────────────┐
│  NotifierService                       │
├────────────────────────────────────────┤
│ notify_user()                          │
│ broadcast_notification()               │
│ send_email()                           │
│ send_sms()                             │
├────────────────────────────────────────┤
│ - Gửi thông báo tức thì               │
│ - Email, SMS notification             │
│ - Broadcast cho nhiều user            │
└────────────────────────────────────────┘
         │
         └─ writes to → Notifications model
```

---

## 8. Quan Hệ Tổng Quát (Entity Relationship)

```
                    ┌─────────────┐
                    │   Role      │
                    └─────────────┘
                          ▲
                          │
                    Users │
                  ┌───────┼───────┐
                  │       │       │
            created│       │       │ manage
             tickets│      │       │
                  │       │       │
        ┌─────────▼┐ ┌──▼───────┐
        │Support   │ │Device    │
        │Ticket    │ │  ┌─────┐ │
        └──────────┘ │  │Type │ │
             │       │  └─────┘ │
             │       │       ▲   │
             │ msg   │       │   │
             │       │  parent│  │
        ┌────▼─────┐ │       │   │
        │Support   │ │  ┌────┴───┴─────┐
        │Message   │ │  │    Device    │
        └──────────┘ │  └──────────────┘
                     │    │       │
              ┌──────┘    │       │
              │      sensor│      │ images
              │         ┌──▼──┐  ┌──▼────┐
              │ belongs │Read │  │  Img  │
              │         └─────┘  └───────┘
              │                      │
              │                 detection
              │                 ┌────▼────┐
              │                 │Detection │
              │                 └────┬────┘
              │                      │
              │                   disease
              │                    ┌──▼──┐
              │                    │Dis- │
              │                    │ease │
              │                    └─────┘
              │
         ┌────▼──────────┐
         │Notifications  │
         └───────────────┘

              ┌────────────┐
              │  Chatbot   │
              └────────────┘
                    │
              details├───► ChatbotDetail
                    │
         ┌──────────┴────────────┐
         │                       │
  ┌──────▼──────┐        ┌──────▼────────┐
  │UserSettings│        │AuthAccount     │
  └─────────────┘        └────────────────┘

```

---

## 9. Data Flow: Quá Trình Phát Hiện Bệnh

```
┌────────────────────────────────────────────────────────────────┐
│              DISEASE DETECTION DATA FLOW                        │
└────────────────────────────────────────────────────────────────┘

User/Camera
    │
    ├─ Upload image
    │
    ▼
┌──────────────────────────┐
│ routes_detect.py         │
│ POST /api/v1/detect      │
└──────────────────────────┘
    │
    ├─ Validate token (AuthJWTService)
    │
    ├─ Check rate limit (DetectLimitService)
    │
    ▼
┌──────────────────────────┐
│ DetectService            │
│ process_detection()      │
└──────────────────────────┘
    │
    ├─ save_image_to_disk()
    │   └─ /media/detections/YYYY/MM/DD/...
    │
    ├─ InferenceService.predict_bytes()
    │   │
    │   └─ YoloDetector.predict_bytes()
    │       ├─ Load model (ml/exports/v1.0/best.pt)
    │       ├─ Process image
    │       └─ Return detections with bbox
    │
    ├─ Map YOLO labels → VN_LABELS
    │
    ├─ ensure_disease() → Disease model
    │
    ▼
┌──────────────────────────┐
│ Detection model          │
│ - img_id: FK             │
│ - disease_id: FK         │
│ - confidence: Decimal    │
│ - bbox: JSON             │
│ - review_status: pending │
└──────────────────────────┘
    │
    ├─ Save to database
    │
    ├─ NotifierService
    │   └─ send_notification() → Users
    │
    ▼
┌──────────────────────────┐
│ Response to Client       │
│ {                        │
│   "detection_id": id,    │
│   "disease": "Lá bưởi...",
│   "confidence": 0.92,    │
│   "bbox": {x, y, w, h},  │
│   "treatment": "...",    │
│   "image_url": "/media/...",
│ }                        │
└──────────────────────────┘
```

---

## 10. Architecture Layers

```
┌──────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                    │
│  (FastAPI Routers - routes_*.py)                        │
├──────────────────────────────────────────────────────────┤
│                     API Endpoints                        │
│  /api/v1/detect, /api/v1/auth, /api/v1/devices, etc.   │
└──────────────────────────────────────────────────────────┘
                            ▲
                            │
┌──────────────────────────────────────────────────────────┐
│                    BUSINESS LOGIC LAYER                  │
│  (Services - services/*.py)                             │
├──────────────────────────────────────────────────────────┤
│ DetectService, DeviceService, AuthJWTService, etc.      │
└──────────────────────────────────────────────────────────┘
                            ▲
                            │
┌──────────────────────────────────────────────────────────┐
│                     DATA LAYER                           │
│  (Schemas & Models - schemas/*.py, models/*.py)         │
├──────────────────────────────────────────────────────────┤
│ SQLAlchemy ORM Models + Pydantic Schemas                │
└──────────────────────────────────────────────────────────┘
                            ▲
                            │
┌──────────────────────────────────────────────────────────┐
│                   DATABASE LAYER                         │
│  (MySQL/PostgreSQL)                                      │
├──────────────────────────────────────────────────────────┤
│ Database Tables & Relationships                         │
└──────────────────────────────────────────────────────────┘
```

---

## 11. Technology Stack Mapping

| Layer | Technology | Role |
|-------|-----------|------|
| **Framework** | FastAPI | RESTful API server |
| **Database ORM** | SQLAlchemy | Model → Database mapping |
| **Validation** | Pydantic | Request/Response schema |
| **AI/ML** | YOLOv8 (Ultralytics) | Object detection |
| **Authentication** | PyJWT | Token-based auth |
| **Scheduler** | APScheduler | Auto-detection scheduling |
| **External APIs** | OpenWeather, OpenAI/LLM | Weather & NLP |
| **File Storage** | Local filesystem | Image storage |
| **Streaming** | HLS | Video streaming |
| **Database** | MySQL/PostgreSQL | Persistent storage |

---

## 12. Key Relations Summary

```
Users
├── 1:1 → Role (admin, support, viewer, support_admin)
├── 1:1 → UserSettings
├── 1:* → Device
├── 1:* → AuthAccount (phone, Google, Facebook, email)
├── 1:* → Img (uploaded/captured images)
├── 1:* → Notification (received)
├── 1:* → Notification (sent as sender)
├── 1:* → SupportTicket
├── 1:* → Chatbot
└── 1:* → SupportMessage

Device
├── *:1 → Users
├── *:1 → DeviceType
├── 1:* → SensorReadings
├── 1:* → DeviceLogs
├── 1:* → Img
└── 1:* → Device (self-parent for hierarchical devices)

Img
├── *:1 → Users
├── *:1 → Device
├── 1:* → Detection
└── SourceType: camera | upload

Detection
├── *:1 → Img
├── *:1 → Disease
├── ReviewStatus: pending, approved, rejected
└── Contains: confidence, bbox (JSON), model_version

Disease
└── 1:* → Detection

SupportTicket
├── *:1 → Users
├── Status: processing, processed
└── 1:* → SupportMessage

SupportMessage
├── *:1 → SupportTicket
└── *:1 → Users (sender)

Notification
├── *:1 → Users (recipient)
├── *:1 → Users (sender, nullable)
└── *:0,1 → Notification (resend_of)

Chatbot
├── *:1 → Users
├── Status: active, ended
└── 1:* → ChatbotDetail

ChatbotDetail
├── Question: Text
└── Answer: Text
```

---

## 13. Core Service Dependencies

```
┌─────────────────────────────────────────────────────────┐
│                   Core Dependencies                      │
└─────────────────────────────────────────────────────────┘

AuthJWTService
├─ Uses: settings (JWT_SECRET, JWT_ALG)
└─ Dependency: routes (all protected endpoints)

DeviceService
├─ Uses: DeviceType, Img, Detection, Disease models
└─ Dependency: routes_devices, routes_detection_history

DetectService
├─ Uses: Img, Detection, Disease models
├─ Uses: InferenceService
└─ Dependency: routes_detect

InferenceService
├─ Uses: YoloDetector
├─ Uses: VN_LABELS mapping
└─ Dependency: DetectService

NotifierService
├─ Uses: Notification model
├─ Uses: Firebase, Email service
└─ Dependency: All services that need to notify

DashboardService
├─ Uses: Device, SensorReadings, Detection models
└─ Dependency: routes_dashboard

SchedulerService
├─ Uses: APScheduler
├─ Uses: AutoDetectionService
└─ Dependency: app startup

PermissionService
├─ Uses: Role, Users models
└─ Dependency: routes (authorization)
```

---

## 14. Enum Types Reference

```
UserStatus: active | inactive
RoleType: support | viewer | admin | support_admin
DeviceStatus: active | maintain | inactive
DeviceTypeStatus: active | inactive
DeviceEventType: online | offline | error | maintenance
SensorStatus: ok | error | missing
SourceType: camera | upload
ReviewStatus: pending | approved | rejected
ChatbotStatus: active | ended
TicketStatus: processing | processed
Provider: sdt | gg | fb | email
```

---

**Cập nhật lần cuối**: 24/12/2025
**Tác giả**: Phân tích từ codebase hệ thống AI Phát Hiện Bệnh Cây Trồng

