# Ghi nhớ kỹ thuật — Thiệp Cưới Sáng & Thu

## Cấu trúc file
| File | Mô tả |
|---|---|
| `nha_trai.html` | Thiệp bản đầy đủ (clone từ index_2.html), dùng cho nhà trai |
| `nha_gai.html` | Thiệp bản đầy đủ, dùng cho nhà gái |
| `index_2.html` | File làm việc chính |
| `index.html` | File gốc ban đầu (đã có nhiều thay đổi) |
| `wedding-config.json` | File cấu hình tham chiếu (không được load trực tiếp) |

---

## Cơ chế đổi ảnh (`triggerMobilePhotoPicker`)

### Cách gọi từ HTML (nút đổi ảnh):
```html
<button type="button"
  onclick="triggerMobilePhotoPicker({ type: 'TEN_TYPE' })"
  class="mobile-edit-badge absolute inset-0 bg-espresso/55 backdrop-blur-[1px]
         flex-col items-center justify-center text-ivory p-3">
  <span class="text-2xl mb-1.5">📷</span>
  <span class="bg-terracotta px-3.5 py-1.5 rounded-full text-xs font-semibold uppercase shadow">
    Chạm Đổi Ảnh Này
  </span>
</button>
```
> **Lưu ý:** `mobile-edit-badge` chỉ hiện khi body có class `mobile-edit-active` (tức khi bật chế độ sửa).

### Các `type` hiện có trong `handleMobileDirectPhotoPicked`:
| type | Hành động |
|---|---|
| `'envelope'` | `WEDDING_CONFIG.envelopePhoto = data` → `syncHeroAndCoverImages()` |
| `'hero'` | `WEDDING_CONFIG.heroMainPhoto = data` → `syncHeroAndCoverImages()` |
| `'ourStory'` | `document.getElementById('ourStoryPhoto').src = data` + lưu `WEDDING_CONFIG.ourStoryPhoto` |
| `'groomPhoto'` | *(cần thêm)* `document.getElementById('groomPhoto').src = data` + `WEDDING_CONFIG.groomPhoto` |
| `'bridePhoto'` | *(cần thêm)* `document.getElementById('bridePhoto').src = data` + `WEDDING_CONFIG.bridePhoto` |
| `'qr'` + `role:'groom'/'bride'` | `WEDDING_CONFIG.bankInfo[role].qrUrl = data` → `initGiftBox()` |
| `'story'` / `'album'` | Mở crop modal trước: `openPhotoCropModal(data, t)` |

### Thêm type mới — pattern chuẩn:
1. Thêm nút HTML vào ảnh (dùng class `mobile-edit-badge`)
2. Thêm case trong `handleMobileDirectPhotoPicked`:
```js
} else if (t.type === 'groomPhoto') {
    const img = document.getElementById('groomPhoto');
    if (img) img.src = compressedDataUrl;
    WEDDING_CONFIG.groomPhoto = compressedDataUrl;
}
```
3. Restore trong `syncHeroAndCoverImages()`:
```js
const groomImg = document.getElementById('groomPhoto');
if (groomImg && WEDDING_CONFIG.groomPhoto) groomImg.src = WEDDING_CONFIG.groomPhoto;
```

---

## Nút Crop ảnh đã upload (✂️)
```html
<button type="button"
  onclick="triggerPhotoCropRetrigger('story', ${idx}, 'photo')"
  class="crop-retrigger-btn absolute top-2 right-2 z-10 w-8 h-8
         rounded-full bg-champagne/90 text-espresso text-sm
         items-center justify-center shadow">✂️</button>
```
> `crop-retrigger-btn` chỉ hiện khi edit active.

---

## Cấu trúc ảnh giống album (bo viền)
```html
<div class="relative bg-white p-2 rounded-2xl shadow-md border border-sand overflow-hidden">
  <div class="relative aspect-[4/5] rounded-xl overflow-hidden bg-champagne">
    <img id="ID_ANH" src="URL" alt="..." class="w-full h-full object-cover">
    <!-- nút đổi ảnh mobile-edit-badge ở đây -->
    <!-- nút crop-retrigger-btn ở đây -->
  </div>
</div>
```

---

## `syncHeroAndCoverImages()` — dòng ~2697 trong nha_trai.html
Hàm này restore ảnh từ `WEDDING_CONFIG` vào DOM khi tải trang hoặc sau khi export/import.  
**Thêm ảnh mới → phải thêm vào đây.**

---

## Vị trí section Chú Rể / Cô Dâu (nha_trai.html)
- Dòng ~484: `familyAnnounceLine` → 3 cột Chú Rể | Trái tim | Cô Dâu
- Dòng ~516: đóng `</div>` của 3 cột
- Dòng ~517: đóng `</div>` div mt-7
- Dòng ~518: đóng `</div>` section B
- Dòng ~521: đóng `</section>` → Tiếp theo là section C (Love Story)

---

## `mobile-edit-badge` vs `crop-retrigger-btn`
- `mobile-edit-badge`: overlay toàn ảnh, hiện khi edit active → dùng để **đổi ảnh mới**
- `crop-retrigger-btn`: nút nhỏ góc phải trên, hiện khi edit active → dùng để **căn lại góc ảnh đã có**

---

## Export file
- Hàm `exportSingleSelfContainedHtml()` clone DOM, reset panel QR, reset nút QR text về "Mở", ghi `WEDDING_CONFIG` mới vào giữa markers `/*__WEDDING_CONFIG_START__*/` và `/*__WEDDING_CONFIG_END__*/`.
- Ảnh được lưu dưới dạng base64 trong `WEDDING_CONFIG`.

---

## Lỗi thường gặp
| Lỗi | Nguyên nhân | Fix |
|---|---|---|
| Nút QR luôn hiện "Đóng" | HTML hardcode sai text ban đầu | Đổi thành `🎁 Mở Hộp Mừng Cưới (QR)` |
| Sau export nút QR sai | `exportSingleSelfContainedHtml` không reset nút | Thêm `giftBtnClone.innerHTML = '<span>🎁 Mở Hộp...</span>'` |
| `heroInvitePrefix` + tên khách hiện cùng lúc | Nhúng tên mặc định vào prefix | Tách: prefix = "Trân trọng kính mời", guest = "Quý Khách & Gia Đình" |

---

## Cấu trúc thư mục & Route

```
thiep_cuoi_online_by_sang/
├── index.html              → domain.com/
├── nha-trai/
│   └── index.html          → domain.com/nha-trai
├── nha-gai/
│   └── index.html          → domain.com/nha-gai
├── assets/
├── vercel.json             → config cleanUrls
├── nha_trai.html           → file gốc (vẫn giữ để edit)
├── nha_gai.html            → file gốc (vẫn giữ để edit)
└── wedding-config.json
```

> **Workflow:** Sửa `nha_trai.html` → copy vào `nha-trai/index.html` trước khi deploy.

### vercel.json
```json
{ "cleanUrls": true, "trailingSlash": false }
```
- `cleanUrls: true` → tự bỏ `.html` khỏi URL
- `trailingSlash: false` → `domain.com/nha-trai` thay vì `domain.com/nha-trai/`

### Deploy lên Vercel
1. Push lên GitHub
2. Vercel tự detect và deploy
3. URL đẹp: `https://ten-project.vercel.app/nha-trai`

---

## Hàm `saveDirectlyToFile()` — Lưu trực tiếp vào file local

**Yêu cầu:** Chrome hoặc Edge (dùng File System Access API)  
**Có trong:** `nha-trai/index.html`, `nha-gai/index.html`

### Cách hoạt động:
1. Sinh HTML hoàn chỉnh (giống `exportSingleSelfContainedHtml`)
2. Mở hộp thoại chọn file `showSaveFilePicker`
3. Người dùng chọn đúng `index.html` trong thư mục dự án
4. Ghi thẳng vào file → không cần tải về rồi copy

### Vị trí nút trong UI:
- **nha-trai**: Trong `mobilePublishModal` → ngay dưới nút "📦 Tải File index.html Hoàn Chỉnh"
- **nha-gai**: Floating bar góc dưới phải → icon 💾 cạnh nút nhạc

### Lưu ý quan trọng:
- `nha-gai/index.html` không có hệ thống WEDDING_CONFIG → hàm chỉ lưu `document.documentElement.outerHTML` (ảnh base64 đã inline sẵn khi chỉnh sửa)
- `nha-trai/index.html` có WEDDING_CONFIG → hàm sinh HTML sạch như `exportSingleSelfContainedHtml`
- Nếu user nhấn Cancel → `AbortError` → không hiện lỗi, bỏ qua

---

## Crop Modal riêng của `nha-gai/index.html`

File này **không dùng** hệ thống `openPhotoCropModal` của nha-trai. Có crop modal độc lập:

| ID | Vai trò |
|---|---|
| `cropModalNhaGai` | Modal overlay fullscreen |
| `cropVpNhaGai` | Viewport crop (div overflow:hidden) |
| `cropImgNhaGai` | Ảnh được kéo/zoom bên trong viewport |
| `cropZoomNhaGai` | Thanh slider zoom 50–300% |

### Các hàm:
- `openCropModal(src, targetImg)` — mở modal, nhận src base64 + phần tử img đích
- `confirmCropNhaGai()` — canvas crop → gán vào targetImg → đóng modal
- `closeCropModalNhaGai()` — huỷ
- `onCropZoomNhaGai(val)` — slider zoom
- `_initCropEvents()` — bind mouse/touch events (gọi trong DOMContentLoaded)
- `_cropApply()` — apply transform left/top/width/height vào ảnh
- `_cropClamp()` — giới hạn không cho ảnh ra ngoài viewport

### Cơ chế đổi ảnh của nha-gai (khác nha-trai):
- Class `photo-editable` trên các `<img>`
- `togglePhotoReplaceMode()` → toggle `edit-photo-mode` trên body
- `initPhotoReplacementHandler()` → click img → `fileInput.click()` → `FileReader` → `openCropModal()`
- Input file: `<input type="file" id="photoFileInput">`
