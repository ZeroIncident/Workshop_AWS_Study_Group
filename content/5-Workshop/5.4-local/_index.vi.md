---
title: "Bước 1 — Chạy và kiểm tra local"
date: 2024-01-01
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

#### Chuẩn bị source

Dùng mã nguồn hiện hành trên GitHub để thực hành tính năng tải ảnh món ăn. ZIP lưu trữ dùng cho lab gốc và có package.json ở thư mục gốc. Từ thư mục mã nguồn:

```bash
unzip aurel-restaurant-source.zip -d aurel-lab
cd aurel-lab
corepack enable
corepack prepare pnpm@11.25.0 --activate
pnpm install --frozen-lockfile
pnpm setup
pnpm dev
```

Mở http://127.0.0.1:3000. Script setup tạo mật khẩu local và khóa phiên ngẫu nhiên trong .env.local nếu file chưa có. Admin local mặc định admin@aurel.local; đọc mật khẩu trên máy, không gửi lên report. Local dùng FileStore một tiến trình, không phải cơ sở dữ liệu cho nhiều máy chủ.

#### Kiểm tra trước khi deploy

Dừng server dev trước smoke nếu cần giải phóng cổng. Smoke dùng cổng 3100 và dữ liệu tạm riêng.

```bash
pnpm check
pnpm build
pnpm test:smoke
```

check gồm TypeScript, Vitest và bundle Lambda/seed. Test phản ánh luật nghiệp vụ, auth, sức chứa, tìm không dấu và pagination; DynamoDB transaction unit test dùng mock, không tương đương chạy cạnh tranh trên dịch vụ thật.

Kiểm tra bằng tay: tìm "pho" và "phở", chuyển nhóm món, phân trang, gửi form sai/đúng, đăng nhập và đổi trạng thái. Các bước bằng tay này là hướng dẫn thực hiện; chỉ kết quả được ghi tại phần 5.8 mới là bằng chứng đã kiểm tra.

#### Font và hình ảnh

Noto Sans/Noto Serif được nhúng bằng next/font/local với glyph tiếng Việt; fallback Segoe UI/Arial/Georgia. Ảnh WebP nằm trong public/images, nguồn/giấy phép theo catalog. Giao diện không cần gọi Google Fonts.   