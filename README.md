# Bài 3: Sửa lỗi bố cục CSS trên giao diện "Theo dõi lộ trình xe"

> Repository nộp bài: `[Tên Lớp]_[Môn Học]_Session16_Ex03`
> Ràng buộc: **không sửa file mã nguồn gốc**, chỉ đề xuất thay đổi qua tab **Styles** trong DevTools (F12).

## Hiện trạng

Khung **"Danh Sách Trạm"** đang đè lên khung **"Thông Tin Tài Xế"**.

![Giao diện lỗi và DevTools](images/f12-inspect.png)

## Chức năng 1: Định vị phần tử bằng Inspect Element

1. Nhấn **F12** (hoặc chuột phải vào khung bị lỗi → **Inspect**).
2. Nhấn biểu tượng **Select an element** (mũi tên trong ô vuông, `Ctrl + Shift + C`), rồi click vào khung "Danh Sách Trạm".
3. Tab **Elements** tự nhảy tới thẻ `<div>` tương ứng trong cây DOM (dòng được tô sáng, có đánh dấu `== $0`).
4. Ở tab **Styles**, đọc các rule đang áp lên thẻ này. Trong ảnh có các thuộc tính đáng ngờ:
   - `margin: 34px` (một số dòng bị gạch ngang, tức là bị rule khác ghi đè)
   - `width: 110%` (lớn hơn khung cha)
   - `padding-bottom: 21px`
   - `z-index: 99999` (đẩy khung nổi lên trên tất cả)

## Phần 1. Phân tích: Box Model hiện tại

Mở tab **Computed**, kéo lên đầu sẽ thấy sơ đồ Box Model của phần tử. Với các thông số đọc được từ ảnh F12, sơ đồ hiện tại như sau:

```
┌──────────────────── margin: 34px ─────────────────────┐
│  ┌────────────────── border: 0 ──────────────────┐    │
│  │  ┌──────────── padding ────────────────────┐  │    │
│  │  │  top 0 | right 0 | bottom 21px | left 0 │  │    │
│  │  │  ┌─────────────────────────────────┐    │  │    │
│  │  │  │   content: width = 110% (cha)   │    │  │    │
│  │  │  └─────────────────────────────────┘    │  │    │
│  │  └─────────────────────────────────────────┘  │    │
│  └───────────────────────────────────────────────┘    │
└───────────────────────────────────────────────────────┘
```

| Lớp | Giá trị hiện tại | Nhận xét |
|-----|------------------|----------|
| Content | `width: 110%` | Rộng hơn khung cha 10%, gây tràn ngang |
| Padding | `padding-bottom: 21px` (các cạnh khác 0) | Không đều, nội dung sát mép |
| Border | `0` | Không có viền để phân tách |
| Margin | `34px` | Bị ghi đè nhiều chỗ, không tạo được khoảng cách ổn định giữa hai khung |

**Nguyên nhân chồng lấn:**

- `width: 110%` làm khung rộng quá khung cha, tràn sang vùng của khung bên cạnh.
- Margin/padding khai báo nhiều lần ở nhiều rule (rule bị gạch ngang), kích thước thực tế không như mong muốn.
- `z-index: 99999` khiến khung "Danh Sách Trạm" luôn nổi lên trên "Thông Tin Tài Xế".
- Thiếu `box-sizing: border-box` nên padding và border cộng thêm vào chiều rộng, càng làm khung phình ra.

> Lưu ý: ảnh đề bài chỉ hiện tab **Styles**. Khi nộp, hãy chụp thêm tab **Computed** (có sơ đồ Box Model) của đúng phần tử bạn chọn và thay số liệu ở bảng trên nếu khác.

## Phần 2. Triển khai: thông số CSS mới

Chỉnh trực tiếp trong tab **Styles** (tick/bỏ tick, hoặc sửa giá trị tại dòng thuộc tính), áp lên phần tử "Danh Sách Trạm":

```css
.station-list {            /* thay bằng đúng class của phần tử đã Inspect */
  box-sizing: border-box;  /* padding + border nằm trong width */
  width: 100%;             /* thay cho 110% */
  margin: 0 0 16px 0;      /* chừa khoảng cách phía dưới giữa hai khung */
  padding: 16px;           /* padding đều 4 cạnh thay cho padding-bottom lẻ */
  border: 1px solid #ddd;  /* viền mảnh để tách biệt hai khung */
  position: relative;      /* nằm trong luồng bố cục, không bị đè */
  z-index: auto;           /* bỏ 99999 */
}
```

Nếu hai khung nằm trong cùng một khung cha, có thể thêm cho khung cha để chia khoảng cách đều:

```css
.container {
  display: flex;
  flex-direction: column;
  gap: 16px;
}
```

**Kết quả mong đợi:** hai khung nằm riêng biệt, không chồng lên nhau, cách nhau 16px, không còn tràn ngang.

## Phần 3. Kiểm thử

**Giả định:** đã đổi Margin nhưng phần tử vẫn không di chuyển. Cần kiểm tra xem có thuộc tính nào đang ghi đè ở tầng Browser Engine không:

1. **Tab Computed → ô Filter, gõ `margin`:** bấm mũi tên cạnh giá trị để xem rule nào đang thắng. Nếu giá trị không phải thứ vừa sửa thì có rule khác ghi đè.
2. **Tab Styles, tìm dòng bị gạch ngang:** dòng bị gạch là rule đã bị thua. Rule thắng thường có độ ưu tiên (specificity) cao hơn, nằm sau, hoặc có `!important`.
3. **Kiểm tra `position`:** nếu là `absolute` / `fixed`, phần tử bị lấy ra khỏi luồng bố cục và vị trí do `top`, `left`, `right`, `bottom` quyết định, đổi margin sẽ gần như không thấy tác dụng.
4. **Kiểm tra `float`, `transform: translate(...)`:** các thuộc tính này dịch chuyển hoặc đặt lại vị trí phần tử độc lập với margin.
5. **Kiểm tra `display`:** nếu phần tử là `inline`, margin trên/dưới không có tác dụng. Cần đổi sang `block` hoặc `inline-block`.
6. **Kiểm tra inline style** (`element.style` ở đầu tab Styles): inline style có độ ưu tiên cao hơn rule trong file CSS.
7. **Kiểm tra margin collapsing:** margin dọc của hai khối liền kề có thể gộp lại thay vì cộng dồn.

Sau mỗi bước, quan sát lại giao diện. Khi hai khung tách rời, không còn chồng lấn thì lỗi đã được khắc phục.

## Quy định nộp bài

- Đưa bản phân tích này và hình ảnh minh họa (F12) lên GitHub, đặt tên repository theo định dạng `[Tên Lớp]_[Môn Học]_Session16_Ex03`.
- Nộp đường dẫn repository lên hệ thống Portal.
