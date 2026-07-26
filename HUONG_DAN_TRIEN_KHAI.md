# HƯỚNG DẪN TRIỂN KHAI QUIZ "THẬT HAY GIẢ?"

## 📂 File bạn có

| File | Mô tả |
|------|--------|
| `quiz-that-hay-gia.html` | File quiz hoàn chỉnh — chỉ cần 1 file này |

---

## 🚀 CÁCH 1: Host trên SharePoint (Khuyên dùng cho nội bộ)

### Bước 1: Upload file
1. Vào SharePoint site của phòng/ban
2. Tạo thư mục, ví dụ: `ATTT-Quiz`
3. Upload file `quiz-that-hay-gia.html` vào thư mục

### Bước 2: Lấy link chia sẻ
1. Click chuột phải vào file → **Share** → **Copy link**
2. Hoặc mở file → copy URL trên thanh địa chỉ
3. Gửi link này lên Viva Engage để mọi người vào làm

### Lưu ý SharePoint
- Nếu SharePoint block file HTML: đổi tên thành `.aspx` hoặc dùng SharePoint Pages embed
- Nếu bị block JavaScript: dùng Cách 2 hoặc Cách 3

---

## 🚀 CÁCH 2: Host trên GitHub Pages (MIỄN PHÍ, đơn giản nhất)

### Bước 1: Tạo repository
1. Vào https://github.com → đăng nhập → New repository
2. Đặt tên: `quiz-attt` → chọn **Public** → Create

### Bước 2: Upload file
1. Click **Add file** → **Upload files**
2. Upload `quiz-that-hay-gia.html` → đổi tên thành `index.html`
3. Commit changes

### Bước 3: Bật GitHub Pages
1. Vào **Settings** → **Pages**
2. Source: chọn **Deploy from a branch**
3. Branch: chọn `main` → `/ (root)` → Save
4. Đợi 1-2 phút, link sẽ xuất hiện:
   `https://<username>.github.io/quiz-attt/`

### Bước 4: Chia sẻ link
- Gửi link lên Viva Engage
- Mọi người click vào là làm bài được ngay trên trình duyệt

---

## 🚀 CÁCH 3: Mở trực tiếp (Demo nhanh)
- Double-click file `quiz-that-hay-gia.html` → mở trong trình duyệt
- Hoạt động đầy đủ, nhưng bảng xếp hạng chỉ lưu trên máy local

---

## 📊 THIẾT LẬP BẢNG XẾP HẠNG TẬP TRUNG (Google Sheets)

> Mặc định quiz lưu điểm vào localStorage (mỗi máy riêng).
> Để **tổng hợp kết quả TẤT CẢ mọi người về 1 chỗ**, làm theo hướng dẫn sau:

### Bước 1: Tạo Google Sheet
1. Vào https://sheets.google.com → tạo Sheet mới
2. Đặt tên: `Quiz ATTT - Kết quả`
3. Ở hàng 1, nhập header:

| A | B | C | D | E | F | G |
|---|---|---|---|---|---|---|
| name | dept | score | total | pct | time | date |

### Bước 2: Tạo Google Apps Script
1. Trong Google Sheet → menu **Extensions** → **Apps Script**
2. Xóa hết code mặc định, paste đoạn code sau:

```javascript
function doPost(e) {
  try {
    var data = JSON.parse(e.postData.contents);
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    sheet.appendRow([
      data.name,
      data.dept,
      data.score,
      data.total,
      data.pct,
      data.time,
      data.date
    ]);
    return ContentService.createTextOutput(
      JSON.stringify({status: "ok"})
    ).setMimeType(ContentService.MimeType.JSON);
  } catch(err) {
    return ContentService.createTextOutput(
      JSON.stringify({status: "error", message: err.toString()})
    ).setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet(e) {
  var action = e.parameter.action;
  if (action === "get") {
    var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    var data = sheet.getDataRange().getValues();
    var headers = data[0];
    var result = [];
    for (var i = 1; i < data.length; i++) {
      var row = {};
      for (var j = 0; j < headers.length; j++) {
        row[headers[j]] = data[i][j];
      }
      result.push(row);
    }
    result.sort(function(a, b) {
      return b.score - a.score || a.time - b.time;
    });
    return ContentService.createTextOutput(
      JSON.stringify(result)
    ).setMimeType(ContentService.MimeType.JSON);
  }
}
```

3. Lưu file (Ctrl+S), đặt tên project: `Quiz API`

### Bước 3: Deploy Apps Script
1. Click **Deploy** → **New deployment**
2. Type: chọn **Web app**
3. Description: `Quiz ATTT API`
4. Execute as: **Me**
5. Who has access: **Anyone** (hoặc **Anyone within [org]** nếu dùng Google Workspace)
6. Click **Deploy** → **Authorize access** → chọn tài khoản → Allow
7. Copy **Web app URL** (dạng `https://script.google.com/macros/s/AKfyc.../exec`)

### Bước 4: Kết nối Quiz với Google Sheet
1. Mở file `quiz-that-hay-gia.html` bằng Notepad/VS Code
2. Tìm dòng:
```javascript
const SHEET_URL = "";
```
3. Paste URL vào:
```javascript
const SHEET_URL = "https://script.google.com/macros/s/AKfyc.../exec";
```
4. Lưu file và upload lại lên nơi host

### Bước 5: Kiểm tra
1. Mở quiz → làm bài → bấm "Lưu điểm"
2. Mở Google Sheet → kiểm tra xem dữ liệu đã ghi vào chưa
3. Bảng xếp hạng sẽ hiển thị kết quả từ tất cả người tham gia

---

## 🖼️ THÊM ẢNH THẬT VÀO QUIZ

Để thay mockup bằng ảnh thật (ảnh chụp email, website, tin nhắn):

### Cách thêm ảnh vào câu hỏi "Hình ảnh AI":
1. Mở file HTML, tìm phần `renderAIImage`
2. Thay dòng `<div class="big">🖼️</div><p>...</p>` bằng:
```html
<img src="ten-file-anh.jpg" style="max-width:100%;border-radius:8px">
```

### Cách thêm câu hỏi mới:
Thêm object mới vào mảng `QUESTIONS`:
```javascript
{
  id: 11,
  category: "Email",      // hoặc "Website", "Tin nhắn", "Hình ảnh AI"
  type: "ai_image",       // email, website, sms, chat, ai_image
  prompt: "Ảnh này là thật hay AI tạo ra?",
  answer: true,           // true = thật, false = giả
  explanation: "Giải thích đáp án...",
  mockup: {
    type: "ai_image",
    description: "Mô tả ảnh...",
    clues: [
      {area: "Chi tiết 1", detail: "Giải thích..."},
    ]
  }
}
```

### Sử dụng ảnh trực tiếp thay vì mockup:
Nếu muốn hiển thị ảnh thay vì mockup HTML, thêm type mới `"image"` và sửa hàm `renderMockup`:
```javascript
// Thêm vào hàm renderMockup:
if (m.type === "image") return `<img src="${m.src}" style="max-width:100%;border-radius:10px;display:block;margin:0 auto">`;

// Và trong QUESTIONS:
mockup: { type: "image", src: "duong-dan-anh.jpg" }
```

---

## ✅ CHECKLIST TRIỂN KHAI

- [ ] Upload file HTML lên nơi host (SharePoint/GitHub Pages)
- [ ] Tạo Google Sheet + Apps Script (nếu cần BXH tập trung)
- [ ] Điền SHEET_URL vào file HTML
- [ ] Test: làm bài thử → kiểm tra dữ liệu ghi vào Sheet
- [ ] Thay mockup bằng ảnh thật (nếu có)
- [ ] Đăng link lên Viva Engage kèm bài giới thiệu
- [ ] Sau 1 tuần: xuất Google Sheet → tổng hợp báo cáo
