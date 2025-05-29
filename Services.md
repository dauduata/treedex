Dưới đây là mô tả chi tiết từng **Backend Service** trong hệ thống, bao gồm kiến trúc, luồng xử lý và công nghệ áp dụng:

---

### 🌿 **1. Plant Catalog Service**  
**Mục đích**: Quản lý thông tin cây cảnh, xử lý tra cứu và tìm kiếm.  

#### **Kiến trúc Service**  
```plaintext
[API Layer] (REST/GraphQL)  
  │  
  ├─ [Controller] → Xử lý HTTP request (GET /plants/{id}, POST /plants/search)  
  │  
  ├─ [Service Layer] → Logic nghiệp vụ (validate, transform data)  
  │  
  ├─ [Repository Layer] → Giao tiếp với DB (PostgreSQL) và Elasticsearch  
  │  
  └─ [Cache Layer] (Redis) → Cache kết quả phổ biến  
```

#### **API Endpoints Chính**  
| Endpoint                  | Method | Mô tả                           |  
|---------------------------|--------|---------------------------------|  
| `/plants`                 | GET    | Danh sách cây (filter, pagination) |  
| `/plants/{id}`            | GET    | Chi tiết cây (kèm care guide)   |  
| `/plants/search`          | POST   | Full-text search (Elasticsearch)|  
| `/plants/categories`      | GET    | Lấy danh mục phân loại          |  

#### **Luồng Xử Lý Điển Hình**  
1. **Search Plants**  
   ```plaintext
   Client → GET /plants?q="lưỡi hổ"  
     → Service kiểm tra cache Redis  
     → Nếu không có → Query Elasticsearch  
     → Trả về kết quả dạng:  
       {  
         "plants": [{...}],  
         "related_categories": ["nội thất"]  
       }  
   ```

2. **Get Plant Details**  
   ```plaintext
   Client → GET /plants/abc123  
     → Service gọi PlantDB + join bảng `plant_care_guides`  
     → Kết hợp ảnh từ Blob Storage (Presigned URL)  
     → Response:  
       {  
         "plant_id": "abc123",  
         "images": ["https://bucket.com/abc123/primary.jpg"],  
         "care_guide": {...}  
       }  
   ```

#### **Công Nghệ**  
- **Ngôn ngữ**: Node.js (Express) / Python (FastAPI)  
- **Database**: PostgreSQL + Elasticsearch  
- **Cache**: Redis (TTL 1h cho trending plants)  
- **Đồng bộ DB-ES**: Logstash hoặc Debezium (CDC)  

---

### 📸 **2. Image Recognition Service**  
**Mục đích**: Nhận diện cây cảnh từ ảnh upload hoặc camera.  

#### **Kiến trúc AI Pipeline**  
```plaintext
[API Gateway] → [Image Preprocessing] → [AI Model Inference] → [Post-processing]  
```

#### **Luồng Xử Lý**  
1. **Client Upload Ảnh**  
   ```plaintext
   Client → POST /recognize (form-data: image)  
     → Service:  
       1. Resize ảnh → 224x224px (chuẩn input model)  
       2. Chuyển đổi định dạng → Tensor (RGB)  
       3. Gọi Model (TensorFlow Serving) → Predict  
       4. Map kết quả (class ID) → plant_id trong DB  
     → Response:  
       {  
         "plant_id": "xyz456",  
         "confidence": 0.92,  
         "scientific_name": "Monstera deliciosa"  
       }  
   ```

2. **Model Training**  
   ```plaintext
   - Sử dụng Pretrained Model (ResNet50) → Fine-tuning với dataset cây cảnh  
   - Dataset cần: 50+ ảnh/cây (đa góc, bệnh, lá non/g già)  
   - Label: Gán bằng plant_id từ PostgreSQL  
   ```

#### **Công Nghệ**  
- **AI Framework**: TensorFlow/Keras (Python)  
- **Model Deployment**: TensorFlow Serving (gRPC)  
- **Optimization**: Quantization (giảm kích thước model cho mobile)  
- **Scaling**: Kubernetes (Horizontal Pod Autoscaler)  

---

### 🧠 **3. Recommendation Engine**  
**Mục đích**: Gợi ý cây phù hợp dựa trên lịch sử người dùng và điều kiện môi trường.  

#### **Kiến trúc Hệ Thống**  
```plaintext
[Input] → [Feature Extraction] → [Recommendation Algorithm] → [Ranking] → [Output]  
```

#### **Các Loại Gợi Ý**  
1. **Dựa trên Người Dùng (Collaborative Filtering)**  
   ```plaintext
   - Phân tích: "Người thích cây A cũng thích cây B"  
   - Data: User favorites, ratings từ Community  
   - Model: Matrix Factorization (PyTorch)  
   ```

2. **Dựa trên Môi Trường**  
   ```plaintext
   - Input: Vị trí (GPS) → Lấy weather data (API)  
   - Logic:  
     IF temperature > 30°C → Gợi ý cây chịu nhiệt (Xương rồng)  
     IF humidity > 70% → Gợi ý cây ưa ẩm (Dương xỉ)  
   ```

3. **Dựa trên Hình Ảnh (Content-Based)**  
   ```plaintext
   - So sánh embedding vector từ AI Model  
   - VD: Ảnh người dùng chụp → Tìm cây có vector tương đồng  
   ```

#### **API Endpoint**  
```plaintext
POST /recommend  
Request: { "user_id": "123", "location": "10.123,106.456" }  
Response: {  
  "plants": [  
    { "plant_id": "789", "match_score": 0.87 },  
    { "plant_id": "456", "match_score": 0.82 }  
  ]  
}  
```

#### **Công Nghệ**  
- **Ngôn ngữ**: Python (scikit-learn, PyTorch)  
- **Data Pipeline**: Apache Spark (xử lý batch)  
- **Realtime**: Redis (lưu user preferences)  

---

### 🔗 **Kết Nối Giữa Các Services**  
```plaintext
1. Image Recognition → Plant Catalog:  
   - Gọi GET /plants/{id} để lấy metadata sau khi nhận diện.  

2. Recommendation Engine → Weather API:  
   - Gọi OpenWeatherMap API để lấy humidity/temperature.  

3. Plant Catalog → Elasticsearch:  
   - Đồng bộ dữ liệu khi có thay đổi (qua message queue).  
```

---

### 📊 **Giám Sát & Tối Ưu**  
- **Logging**: ELK Stack (theo dõi latency của AI model).  
- **Monitoring**: Prometheus (số lượng request/error rate).  
- **Tối ưu**:  
  - Plant Catalog: Thêm CDN cho ảnh tĩnh.  
  - Image Recognition: Sử dụng GPU instance cho model inference.  
