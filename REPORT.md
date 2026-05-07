# Lab 21 — Evaluation Report

**Học viên**: Nguyễn Duy Hiếu — 2A202600153
**Ngày nộp**: 2026-05-07
**Submission option**: A (Lightweight ZIP) - Đã bao gồm adapter r16 trong thư mục results.

## 1. Setup
- **Base model**: `unsloth/Qwen2.5-3B-bnb-4bit`
- **Dataset**: `5CD-AI/Vietnamese-alpaca-gpt4-gg-translated`, 200 samples (180 train + 20 eval)
- **max_seq_length**: 1024 (p95 = 562, rounded up to power of 2)
- **GPU**: Tesla T4, 16 GB VRAM
- **Training cost**: ~$0.07 (Tổng 12.8 phút @ $0.35/hr)

## 2. Rank Experiment Results

| Rank | Trainable Params | Train Time | Peak VRAM | Eval Loss | Perplexity |
|------|-----------------|------------|-----------|-----------|------------|
| 8    | 1,843,200       | 4.14 min   | 7.22 GB   | 1.5574    | 4.75       |
| 16   | 3,686,400       | 4.51 min   | 6.62 GB   | 1.5161    | 4.55       |
| 64   | 14,745,600      | 4.18 min   | 8.00 GB   | 1.4766    | 4.38       |
| Base | -               | -          | -         | 1.5712*   | 4.81*      |
*\*Số liệu Base ước tính từ log khởi điểm.*

## 3. Loss Curve Analysis
- **Quan sát**: Training loss giảm dần từ ~1.6 xuống ~1.39 sau 3 epochs. Đường loss có răng cưa nhẹ do batch size nhỏ (1), nhưng xu hướng chung là hội tụ tốt.
- **Overfitting**: Không có dấu hiệu overfitting rõ rệt. Eval loss (1.51) cho rank 16 chỉ cao hơn train loss một chút, mô hình vẫn giữ được khả năng generalize tốt trên tập validation.

## 4. Qualitative Comparison (Trích 3/5 ví dụ)

### Example 1
**Prompt**: Giải thích khái niệm machine learning cho người mới bắt đầu.
**Base**: Machine learning là một kỹ thuật trong trí tuệ nhân tạo nhằm giúp máy tự động học... (Ngôn ngữ hơi chung chung).
**Fine-tuned (r=16)**: Machine learning là một phân ngành của AI... giải thích chi tiết hơn và dùng thuật ngữ chuẩn xác hơn.
**Nhận xét**: Improved. Câu văn mạch lạc và chuyên sâu hơn.

### Example 2
**Prompt**: Viết đoạn code Python tính số Fibonacci thứ n.
**Base**: Trả về đoạn code tìm subarray có tổng lớn nhất (Sai hoàn toàn nội dung).
**Fine-tuned (r=16)**: Trả về đoạn code Python dùng đệ quy chính xác cho Fibonacci.
**Nhận xét**: Significantly Improved. Mô hình đã học được cách tuân thủ instruction chính xác.

### Example 3
**Prompt**: Liệt kê 5 nguyên tắc thiết kế UI/UX.
**Base**: Trả về Tiếng Trung (1. 用户中心原则...).
**Fine-tuned (r=16)**: Trả về Tiếng Việt chuẩn (1. Chú ý đến trải nghiệm người dùng...).
**Nhận xét**: Improved. Giải quyết triệt để vấn đề bị lẫn lộn ngôn ngữ (language drifting).

## 5. Conclusion về Rank Trade-off
Dựa trên thực nghiệm, rank **r=16** cho ROI (Return on Investment) tốt nhất. Mặc dù r=64 cho Perplexity thấp nhất (4.38), nhưng sự cải thiện so với r=16 (4.55) là không quá đột biến trong khi lượng tham số tăng gấp 4 lần và VRAM chiếm dụng cao hơn. Hiện tượng "diminishing returns" bắt đầu xuất hiện rõ rệt sau rank 16. Nếu deploy production, tôi sẽ chọn rank 16 vì nó tiết kiệm tài nguyên mà vẫn đảm bảo chất lượng phản hồi Tiếng Việt vượt trội so với mô hình gốc.

## 6. What I Learned
- Hiểu cách LoRA giúp cập nhật tri thức mới (Tiếng Việt) mà không cần cập nhật toàn bộ trọng số mô hình.
- Biết cách sử dụng Unsloth để tối ưu tốc độ fine-tuning trên phần cứng hạn chế như T4.
- Nhận ra tầm quan trọng của việc chọn Rank: Rank cao chưa chắc đã tốt nếu dataset nhỏ, dễ dẫn đến lãng phí tài nguyên.
