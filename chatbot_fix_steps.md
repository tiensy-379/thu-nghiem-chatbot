# Bước 1: Sửa nhớ ngữ cảnh tour cho câu hỏi nối tiếp (giá/chương trình)

## Mục tiêu
- Khi user vừa hỏi 1 tour cụ thể, các câu nối tiếp như `giá tour`, `chương trình tour`, `lịch trình` phải bám đúng tour gần nhất.
- Không rơi về trả lời chung chung nếu cuộc hội thoại đã có ngữ cảnh tour.

## File cần sửa
- `app.py`

## Cách sửa (Tìm kiếm thay thế / chèn) — định vị rõ ràng, tránh nhầm

### A) Chèn logic đọc tour từ memory (chèn 1 lần duy nhất)

**Điểm neo để tìm (neo 2 dòng liên tiếp):**
```python
# ================== AI-POWERED CONTEXT ANALYSIS ==================
message_lower = user_message.lower()
```

**Chèn NGAY BÊN DƯỚI `message_lower = user_message.lower()` đoạn sau:**
```python
# CONTEXT MEMORY (follow-up):
# Nếu user đang hỏi nối tiếp về giá/chương trình/lịch trình,
# và lượt này chưa match được tour mới thì dùng tour gần nhất trong session.
followup_keywords = [
    'giá tour', 'giá', 'chương trình', 'lịch trình', 'chi tiết tour', 'tour này'
]
is_followup_tour_question = any(k in message_lower for k in followup_keywords)

# Lưu ý: tour_indices đã được khởi tạo [] ở đầu hàm.
if is_followup_tour_question and not tour_indices:
    last_tour_idx = getattr(context, 'current_tour', None)
    if isinstance(last_tour_idx, int) and last_tour_idx in TOURS_DB:
        tour_indices = [last_tour_idx]
        logger.info(f"🧠 Reuse context.current_tour={last_tour_idx} for follow-up")
```

---

### B) Thay block cập nhật context cuối hàm để lưu thêm thời điểm

**Điểm neo để tìm (nguyên block hiện tại):**
```python
# Cập nhật tour context nếu có tour được đề cập
if tour_indices and len(tour_indices) > 0:
    context.current_tour = tour_indices[0]
    tour = TOURS_DB.get(tour_indices[0])
    if tour:
        context.last_tour_name = tour.name
```

**Thay TOÀN BỘ block trên bằng:**
```python
# Cập nhật tour context nếu có tour được đề cập
if tour_indices and len(tour_indices) > 0:
    context.current_tour = tour_indices[0]
    context.current_tour_updated_at = datetime.utcnow().isoformat()
    tour = TOURS_DB.get(tour_indices[0])
    if tour:
        context.last_tour_name = tour.name
```

## Test nhanh sau khi sửa (manual)
1. Hỏi: `bạn có những tour nào?`
2. Hỏi tiếp: `chương trình tour Mưa Đỏ và Trường Sơn – Hành Trình Khát Vọng`
3. Hỏi tiếp: `giá tour`

**Kỳ vọng:** câu 3 phải trả theo tour ở câu 2, không trả bảng giá chung.
