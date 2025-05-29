Dưới đây là thiết kế chi tiết **kiến trúc database** cho từng loại database trong ứng dụng bách khoa cây cảnh, tối ưu cho từng nghiệp vụ cụ thể:

---

### 🌿 **1. PLANTDB (PostgreSQL - Structured Data)**
**Mục đích**: Lưu trữ dữ liệu có cấu trúc về cây cảnh, quan hệ phức tạp.  
**Các bảng chính**:

#### **a. Bảng `plants` (Core Data)**
```sql
CREATE TABLE plants (
    plant_id UUID PRIMARY KEY,
    common_name VARCHAR(100) NOT NULL,          -- Tên phổ thông (VD: "Cây Lưỡi Hổ")
    scientific_name VARCHAR(100),               -- Tên khoa học (VD: "Sansevieria trifasciata")
    family VARCHAR(50),                         -- Họ thực vật (VD: "Asparagaceae")
    description TEXT,                           -- Mô tả chi tiết
    toxicity_level VARCHAR(20),                 -- Độc tính (NULL/an toàn, "low", "high")
    created_at TIMESTAMPTZ DEFAULT NOW()
);
```

#### **b. Bảng `plant_care_guides` (Hướng dẫn chăm sóc)**
```sql
CREATE TABLE plant_care_guides (
    guide_id UUID PRIMARY KEY,
    plant_id UUID REFERENCES plants(plant_id),
    water_frequency VARCHAR(50),                -- "1 lần/tuần"
    light_requirements VARCHAR(50),             -- "Ánh sáng gián tiếp"
    humidity_range VARCHAR(50),                 -- "40-60%"
    temperature_range VARCHAR(50),              -- "18-25°C"
    fertilizer TEXT                             -- Hướng dẫn bón phân
);
```

#### **c. Bảng `plant_diseases` (Bệnh thường gặp)**
```sql
CREATE TABLE plant_diseases (
    disease_id UUID PRIMARY KEY,
    plant_id UUID REFERENCES plants(plant_id),
    name VARCHAR(100),                          -- Tên bệnh (VD: "Thối rễ")
    symptoms TEXT,                              -- Triệu chứng
    treatment TEXT                              -- Cách xử lý
);
```

#### **d. Bảng `plant_images` (Ảnh minh họa)**
```sql
CREATE TABLE plant_images (
    image_id UUID PRIMARY KEY,
    plant_id UUID REFERENCES plants(plant_id),
    image_url TEXT NOT NULL,                    -- URL lưu trên Blob Storage
    caption VARCHAR(200),                       -- Chú thích ảnh
    is_primary BOOLEAN DEFAULT FALSE            -- Ảnh đại diện
);
```

#### **e. Bảng `categories` (Phân loại)**
```sql
CREATE TABLE categories (
    category_id UUID PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL            -- VD: "Cây nội thất", "Xương rồng"
);

-- Bảng liên kết nhiều-nhiều (plants <-> categories)
CREATE TABLE plant_categories (
    plant_id UUID REFERENCES plants(plant_id),
    category_id UUID REFERENCES categories(category_id),
    PRIMARY KEY (plant_id, category_id)
);
```

**Indexing Strategy**:
```sql
CREATE INDEX idx_plants_common_name ON plants(common_name);
CREATE INDEX idx_plants_scientific_name ON plants(scientific_name);
CREATE INDEX idx_plant_care_guides ON plant_care_guides(plant_id);
```

---

### 🖼️ **2. BLOB STORAGE (AWS S3/Cloudinary)**
**Mục đích**: Lưu trữ ảnh/video (không structured).  
**Cấu trúc thư mục**:
```plaintext
plant-app/
│
├── plants/
│   ├── {plant_id}/
│   │   ├── primary.jpg
│   │   ├── leaf_closeup.jpg
│   │   └── growth_timelapse.mp4
│
└── users/
    ├── {user_id}/
    │   └── avatar.png
```

**Quy tắc**:
- Mỗi cây có thư mục riêng với `plant_id` làm tên.
- Định dạng ảnh: WebP (tối ưu dung lượng).
- Metadata kèm theo: 
  - `uploaded_at`: Thời gian upload.
  - `uploaded_by`: User ID (nếu có).

---

### 👥 **3. USER DATA (Firestore/Firebase - NoSQL)**
**Mục đích**: Lưu dữ liệu người dùng (scalability, unstructured).  
**Collections**:

#### **a. Collection `users`**
```javascript
{
  userId: "abc123",
  email: "user@example.com",
  displayName: "Plant Lover",
  favorites: ["plant_id_1", "plant_id_2"], // Danh sách cây yêu thích
  savedGuides: ["guide_id_1"] // Hướng dẫn đã lưu
}
```

#### **b. Collection `community_posts`**
```javascript
{
  postId: "post123",
  userId: "abc123",
  plantId: "plant_id_1", // Liên kết với cây (optional)
  content: "Cây của tôi bị vàng lá...",
  images: ["url1", "url2"],
  timestamp: "2023-10-20T10:00:00Z",
  likes: 15,
  comments: [
    {
      userId: "user456",
      text: "Thử kiểm tra độ ẩm!",
      timestamp: "2023-10-20T11:30:00Z"
    }
  ]
}
```

**Security Rules** (Firebase):
```javascript
match /users/{userId} {
  allow read, write: if request.auth.uid == userId;
}

match /community_posts/{postId} {
  allow read: if true;
  allow create: if request.auth.uid != null;
  allow update, delete: if request.auth.uid == resource.data.userId;
}
```

---

### 🔍 **4. ELASTICSEARCH (Search Engine)**
**Mục đích**: Tìm kiếm full-text nhanh.  
**Cấu trúc document** (cho cây cảnh):
```json
{
  "id": "plant_id_1",
  "common_name": "Cây Lưỡi Hổ",
  "scientific_name": "Sansevieria trifasciata",
  "family": "Asparagaceae",
  "description": "Cây chịu hạn tốt, lá cứng...",
  "care_tips": "Tưới 1 lần/tuần...",
  "categories": ["nội thất", "dễ trồng"]
}
```

**Analyzers**:
- Sử dụng **vi_analyzer** (nếu cần tìm kiếm tiếng Việt).
- Highlight kết quả tìm kiếm.

---

### ⚡ **5. REDIS (Cache Layer)**
**Mục đích**: Giảm tải database.  
**Cấu trúc key-value**:
```plaintext
# Cache thông tin cây (TTL: 1 giờ)
plant:{plant_id} => JSON(plant_data)

# Cache kết quả tìm kiếm (TTL: 30 phút)
search:{query} => [plant_id_1, plant_id_2]

# Top cây phổ biến (update hàng tuần)
top_plants => [plant_id_3, plant_id_9]
```

---

### 📊 **Sơ Đồ Quan Hệ Tổng Thể**
```plaintext
PostgreSQL (PlantDB) ────┐
                        ├─ Sync → Elasticsearch
Firestore (User Data)   │
                        ├─ Event Streaming → Redis Cache
Blob Storage (Images) ──┘
```

**Cơ chế đồng bộ**:
- PostgreSQL → Elasticsearch: Dùng **Logstash** hoặc CDC (Change Data Capture).
- User Actions → Redis: Pub/Sub pattern.

---

### 💡 **Tối Ưu Hiệu Năng**
1. **PostgreSQL**:
   - Partition bảng `plant_images` theo `plant_id` nếu dữ liệu lớn.
   - Sử dụng materialized views cho báo cáo.
2. **Blob Storage**:
   - CDN (CloudFront) để phân phối ảnh toàn cầu.
3. **Firestore**:
   - Composite indexes cho các query phức tạp (VD: `[userId, timestamp]`).
