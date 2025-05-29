
---

### 📱 **Các Màn Hình Chính** (Mobile UI Wireframe)
#### 1. **Màn hình Chính (Home)**
- **Thành phần**:
  - Thanh tìm kiếm nổi bật (Search Bar).
  - Danh sách cây phổ biến (Horizontal Scroll) với ảnh thumbnail và tên.
  - Categories (Phân loại theo môi trường sống: Indoor, Outdoor, Cây thuốc...).
  - Nút "Nhận diện ảnh" (Camera Icon) ở góc phải.
- **UI Example**:
  ```
  [Search 🔍]
  [Popular Plants]
  ┌───────┐ ┌───────┐
  │Cây Lưỡi Hổ│ │Sen Đá    │
  └───────┘ └───────┘
  [Categories]
  [Indoor] [Outdoor] 
  [Floating Button: 📷]
  ```

#### 2. **Màn hình Chi Tiết Cây (Plant Detail)**
- **Thành phần**:
  - Ảnh lớn (Carousel nhiều góc chụp).
  - Tên cây + tên khoa học.
  - Tab Navigation: 
    - **Thông tin**: Mô tả, họ thực vật, kích thước.
    - **Chăm sóc**: Tưới nước, ánh sáng, độ ẩm.
    - **Bệnh**: Các bệnh thường gặp + cách xử lý.
  - Nút "Thêm vào yêu thích" ♡.
- **UI Example**:
  ```
  [Ảnh Cây]
  [Monstera] 
  [Monstera deliciosa]
  ┌───────────────┐
  │Thông tin │ Chăm sóc │ Bệnh │
  └───────────────┘
  [Mô tả: Cây lá xẻ...]
  [♡ Add to Favorites]
  ```

#### 3. **Màn hình Nhận diện Ảnh (AI Scan)**
- **Thành phần**:
  - Camera Preview (chiếm 80% màn hình).
  - Nút chụp ảnh (to, tròn ở giữa).
  - Nút chọn ảnh từ thư viện.
  - Kết quả nhận diện hiển thị ngay sau khi chụp (dạng pop-up).
- **UI Example**:
  ```
  [Camera Preview]
  [◯] (Nút chụp)
  [📁 Gallery]
  ```

#### 4. **Màn hình Danh sách Yêu thích (Favorites)**
- **Thành phần**:
  - Danh sách dạng lưới (Grid) hoặc list.
  - Mỗi item hiển thị ảnh + tên cây.
  - Nút lọc (Filter) theo loại cây.
- **UI Example**:
  ```
  [My Favorites ♡]
  ┌───────┐ ┌───────┐
  │Cây Kim Tiền│ │Lưỡi Hổ   │
  └───────┘ └───────┘
  ```

#### 5. **Màn hình Cộng đồng (Community)**
- **Thành phần**:
  - Tab: Bài viết mới (New Posts) / Hỏi đáp (Q&A).
  - Nút tạo bài viết (FAB - Floating Action Button).
  - Card bài viết với avatar người dùng, nội dung ngắn.
- **UI Example**:
  ```
  [Community]
  [New Posts │ Q&A]
  ┌─────────────────┐
  │UserA: Cây bị vàng lá...│
  └─────────────────┘
  [✏️ Create Post]
  ```

---

### 🔄 **Sơ Đồ Luồng Hoạt Động (User Flow)**
```plaintext
Start → [Home]
  ├─ [Search] → [Search Results] → [Plant Detail]
  ├─ [AI Scan] → [Camera/Gallery] → [Scan Result] → [Plant Detail]
  ├─ [Categories] → [Category List] → [Plant Detail]
  └─ [Community] → [Post Detail] / [New Post]
```

#### Giải thích flow:
1. **Từ Home**:
   - Người dùng có thể đi đến bất kỳ màn hình con nào (Search, Scan, Categories, Community).
2. **Màn hình Chi Tiết**:
   - Là điểm trung tâm, xuất hiện sau khi chọn cây từ Search/AI Scan/Categories.
3. **Community**:
   - Hoạt động độc lập, tập trung vào tương tác người dùng.

---

### 🎨 **UI/UX Recommendations**
1. **Navigation**:
   - Dùng **Bottom Navigation Bar** với 4 tab: Home, Scan, Favorites, Community.
2. **Consistency**:
   - Màu chủ đạo: Xanh lá (#2E7D32) - gắn với thiên nhiên.
   - Font chữ: Roboto (dễ đọc trên mobile).
3. **Micro-interactions**:
   - Animation khi chuyển tab hoặc thêm vào yêu thích.
4. **Accessibility**:
   - Đảm bảo tỷ lệ tương phản màu sắc đạt WCAG AA.

---

### 📊 **Data Flow for Key Screens**
Ví dụ với **AI Scan**:
```plaintext
1. User mở Camera → [Client] gọi API Gateway.
2. API Gateway → ImageRecognitionService (Microservice).
3. Service xử lý ảnh bằng AI Model → Trả về ID cây.
4. Client nhận ID → Fetch chi tiết từ PlantService.
5. Hiển thị Plant Detail với dữ liệu từ PlantDB.
```

---

### 📱 **Tools Để Thiết Kế Chi Tiết**
- **Wireframe**: Figma, Adobe XD (tham khảo Material Design Guidelines).
- **Prototype**: InVision hoặc Figma Interactive.
- **Assets**: Dùng illustration cây cảnh từ Undraw hoặc Freepik.

Bạn muốn tập trung vào flow cụ thể nào (ví dụ: luồng đăng bài cộng đồng) hoặc cần mình mô tả thêm về UI components? 😊
