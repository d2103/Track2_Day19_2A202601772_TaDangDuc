# Reflection — Lab 19

**Tên:** Tạ Đăng Đức
**Cohort:** A20-K4
**Path đã chạy:** lite (WSL2 / Ubuntu, Python 3.12)

---

## Câu hỏi (≤ 200 chữ)

> Trên golden set 50 queries, mode nào thắng ở loại query nào (`exact` /
> `paraphrase` / `mixed`), và tại sao? Khi nào bạn **không** dùng hybrid
> (i.e. khi nào pure BM25 hoặc pure vector là lựa chọn đúng)?

Precision@10 trung bình: hybrid 78,6% > BM25 77,8% > vector 73,2%.

Theo slice: `exact` BM25 thắng (96,7% so với 88,7% của vector) — query chứa
đúng thuật ngữ trong doc, tín hiệu lexical đủ mạnh. `mixed` hybrid thắng
rõ (100% so với 97,0/98,5%): mỗi retriever bắt được một nửa ý định, RRF cộng
hai thứ hạng lại nên doc nào cả hai cùng thấy sẽ nổi lên đầu.

`paraphrase` là chỗ trái kỳ vọng: vector chỉ 24,0%, **thấp nhất cả ba**. Nguyên
nhân không phải RRF sai mà là chọn model: `bge-small-en-v1.5` được train trên
tiếng Anh, trong khi corpus và query đều tiếng Việt, nên vector không tách được
các cách diễn đạt lại. Cách sửa đúng là đổi sang `bge-m3` (đa ngữ), không phải
chỉnh `k`.

Không dùng hybrid khi: (1) latency ngặt — cùng một server, keyword P50 1,8 ms
còn hybrid 72,1 ms, chênh 40 lần, vì hybrid phải embed query mỗi lần gọi;
(2) query là mã lỗi, SKU, ID — BM25 chính xác hơn và rẻ hơn; (3) embedding
model không phủ ngôn ngữ của corpus, lúc đó nhánh vector chỉ thêm nhiễu như
số liệu `paraphrase` cho thấy.

---

## Điều ngạc nhiên nhất khi làm lab này

Chi phí thật của lab không nằm ở thuật toán mà ở vận hành. Mỗi kernel Jupyter
và mỗi uvicorn còn sống giữ nguyên một `Searcher` với 1000 vector; sau vài giờ
chúng chiếm 7,4/7,6 GB RAM của WSL và máy bắt đầu swap. Dọn sạch chúng kéo
hybrid P99 từ 542 ms xuống 126,5 ms — nhanh 4,3 lần mà không sửa một dòng code
nào. Nhưng vẫn trượt ngưỡng 50 ms: phần còn lại là tốc độ inference ONNX của
máy (embed một query ~83 ms), thứ mà dọn RAM không sửa được.

---

## Bonus challenge

- [ ] Đã làm bonus (xem `bonus/`)
- [ ] Pair work với: _<tên đồng đội nếu có>_
