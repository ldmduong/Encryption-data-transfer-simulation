# Mô Phỏng Hệ Thống Truyền Thông Bảo Mật: Kiến Trúc Lai RSA + AES

[![Python Version](https://img.shields.io/badge/Python-3.9%2B-blue.svg?logo=python&logoColor=white)](https://www.python.org/)
[![Cryptography](https://img.shields.io/badge/Library-Cryptography-green.svg)](https://cryptography.io/)
[![Course](https://img.shields.io/badge/HUST-ET3310--Lý_Thuyết_Mật_Mã-red.svg)](https://hust.edu.vn/)

Dự án mô phỏng hệ thống truyền dữ liệu đa phương tiện (văn bản và hình ảnh) an toàn qua môi trường mạng không tin cậy bằng **Kiến trúc mật mã lai (Hybrid Cryptosystem)** kết hợp giữa **AES-256-CBC** và **RSA-2048 OAEP**, thuộc học phần *ET3310 – Lý thuyết Mật mã* (Đại học Bách Khoa Hà Nội).

---

## 📌 Bối Cảnh & Nguyên Lý Kiến Trúc Lai

Khi truyền tải dữ liệu lớn (ảnh/tập tin) qua mạng lõi viễn thông ở dạng dữ liệu rõ (Cleartext), thông tin dễ bị nghe lén và sửa đổi.
- **RSA (Bất đối xứng):** An toàn cao, giải quyết triệt để bài toán phân phối khóa nhưng tốc độ tính toán rất chậm đối với dữ liệu dung lượng lớn.
- **AES (Đối xứng):** Tốc độ mã hóa khối cực nhanh (lên tới hàng trăm MB/s), nhưng đòi hỏi kênh truyền bảo mật để chia sẻ khóa bí mật.

**Giải pháp kết hợp:**
1. Sử dụng **AES-256-CBC** để mã hóa khối dữ liệu tải trọng (Payload: văn bản/ảnh).
2. Sử dụng **RSA-2048 (padding OAEP)** với Public Key của người nhận để đóng gói (wrap) chính khóa AES đó.
3. Người nhận dùng RSA Private Key để bọc mở khóa AES, sau đó dùng khóa AES để giải mã dữ liệu về nguyên bản.

---

## 🔄 Quy Trình Mô Phỏng 5 Bước

```text
[ Người Gửi ]                                                    [ Người Nhận ]
      │                                                                 │
      ├── 1. Sinh khóa: AES-256 Key (ngẫu nhiên)                        │
      ├── 2. Mã hóa dữ liệu: AES-256-CBC (Data + IV ──> Ciphertext)     │
      ├── 3. Đóng gói khóa: RSA-2048 OAEP(AES Key) ──> Encrypted Key    │
      │                                                                 │
      └── 4. Đóng gói Packet ───────────────────────────────►           │
             [Header | RSA Encrypted Key | IV | Ciphertext]             │
                                                                        │
                                   5. Phân tích gói & Giải mã: ─────────┤
                                   ├── RSA Decrypt(Encrypted Key) ──> Khóa AES
                                   ├── AES-256 Decrypt(Ciphertext) ──> Plaintext
                                   └── Kiểm tra toàn vẹn SHA-256 ──> Khớp 100%
```

---

## 🧪 Kết Quả Thực Nghiệm & Đánh Giá Bảo Mật

### 1. Hiệu Năng & Đo Lường

| Phép toán | Kích thước dữ liệu | Thời gian thực thi trung bình | Ghi chú |
| :--- | :--- | :--- | :--- |
| **AES-256-CBC Encrypt** | ~2.9 KB (ảnh) | ~0.08 ms (Throughput: 36.8 MB/s) | Tuyến tính theo kích thước file |
| **AES-256-CBC Decrypt** | ~2.9 KB (ảnh) | ~0.04 ms (Throughput: 68.8 MB/s) | Phục hồi hoàn chỉnh từng byte |
| **RSA-2048 OAEP Encrypt** | 32 bytes (khóa AES) | ~0.22 ms | Thời gian cố định |
| **RSA-2048 OAEP Decrypt** | 256 bytes (khóa bọc) | ~0.71 ms | Chậm hơn mã hóa do số mũ private $d > e$ |

- **Tính toàn vẹn (Integrity):** Mã băm SHA-256 trước và sau giải mã trùng khớp 100% trên cả file text và file ảnh (kiểm tra pixel-by-pixel không sai lệch bất kỳ điểm ảnh nào).
- **Phân tích Entropy:** Plaintext có entropy ~`7.45 bits/byte`, sau mã hóa AES entropy đạt ~`7.93 bits/byte` (tiệm cận tối đa 8.0 bits/byte — phân bố ngẫu nhiên hoàn toàn).

### 2. Kịch Bản Tấn Công Thử Nghiệm

- **Dùng sai khóa AES:** Dữ liệu giải mã ra chuỗi vô nghĩa, sai mã hash SHA-256.
- **Bit-flip Attack (Sửa 1 byte ciphertext):** SHA-256 thay đổi hoàn toàn, phát hiện xâm phạm dữ liệu ngay lập tức.
- **Giải mã bằng RSA Private Key sai:** Hệ thống báo lỗi xác thực padding `OAEP padding verification failed`, ngăn chặn kẻ tấn công trích xuất khóa AES.

---

## 📁 Cấu Trúc File & Dữ Liệu Đầu Ra

```text
.
├── Mã_hóa_AES_và_RSA_cho_HT_truyền_dữ_liệu.ipynb   # Toàn bộ mã nguồn và thực nghiệm mô phỏng
├── README.md                                      # Tài liệu mô tả dự án
├── test_data/                                     # Dữ liệu kiểm thử đầu vào
│   ├── bao_cao_mat_nhom.txt                       # File văn bản rõ
│   └── anh_doc_quyen_nhom.png                     # File ảnh kiểm thử sinh tự động
└── output_data/                                   # Kết quả sau khi chạy notebook
    ├── encrypted_aes_key.bin                      # Khóa AES 256-bit được bọc bằng RSA (256 bytes)
    ├── text_encrypted.bin / text_decrypted.txt    # Ciphertext và file văn bản giải mã
    ├── image_encrypted.bin / image_decrypted.png  # Ciphertext và file ảnh giải mã
    ├── text_packet_transmitted.bin                # Khung gói tin truyền dẫn hoàn chỉnh
    ├── image_packet_transmitted.bin
    ├── visualization_image_comparison.png         # Trực quan hóa ảnh gốc vs ciphertext vs ảnh phục hồi
    ├── visualization_entropy.png                  # Biểu đồ Histogram & Entropy
    ├── visualization_system_flow.png              # Sơ đồ luồng dữ liệu hệ thống
    └── visualization_performance.png              # Biểu đồ so sánh hiệu năng AES vs RSA
```

---

## 🚀 Hướng Dẫn Cài Đặt & Chạy Mô Phỏng

### 1. Cài đặt môi trường

Cài đặt
