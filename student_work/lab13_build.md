# Ghi chú hoàn thiện Lab 13 Observathon

## 1. Mục tiêu lab

Lab Day-13 Observathon yêu cầu quan sát và tối ưu một agent thương mại điện tử dạng black-box. Agent có thể gặp nhiều lỗi vận hành như tính toán sai, gọi tool sai, lặp tool, lỗi tạm thời từ tool, rò rỉ PII, prompt injection và giảm chất lượng theo session.

Bài làm tập trung vào các file chính trong thư mục `solution/`:

- `config.json`
- `config_practice.json`
- `prompt.txt`
- `wrapper.py`
- `findings.json`
- `findings_with_private_injection.json`
- `examples.json`

## 2. Môi trường đã dùng

Máy Windows gặp lỗi khi chạy binary Windows:

```text
Failed to load Python DLL python312.dll
LoadLibrary: Invalid access to memory location
```

Vì vậy lab được chạy bằng WSL Ubuntu với các binary Linux x64.

Các binary đã dùng:

```text
bin/practice/observathon-sim
bin/public/observathon-sim
bin/public/observathon-score
bin/private/observathon-sim
bin/private/observathon-score
```

API key OpenAI được lưu trong file `.env`. File này chỉ dùng cục bộ và không được commit lên Git.

## 3. Cách nạp API key trong WSL

Chạy trong thư mục lab:

```bash
cd /mnt/d/AI20k/lab13/Day-13-Lab-Observathon
export OPENAI_API_KEY="$(grep '^OPENAI_API_KEY=' .env | cut -d= -f2- | tr -d '\r' | sed 's/^"//;s/"$//')"
```

## 4. Practice phase

Practice v5 dùng mock engine, không cần OpenAI API key. File cấu hình dùng riêng cho practice là `solution/config_practice.json`.

Lệnh chạy:

```bash
./bin/practice/observathon-sim --config solution/config_practice.json --wrapper solution/wrapper.py --out run_output.json
```

Kết quả practice:

```text
20/20 status ok
```

## 5. Public phase

Public v6 dùng real LLM và OpenAI API key.

Chạy public simulator:

```bash
./bin/public/observathon-sim --config solution/config.json --wrapper solution/wrapper.py --out run_output.json --concurrency 8
```

Chấm điểm public:

```bash
./bin/public/observathon-score --run run_output.json --findings solution/findings.json --team local-team --out score.json
```

Kết quả public mới nhất:

```text
120/120 status ok
Public score: 100.0 / 100
Correct: 93 / 120
Diagnosis F1: 1.0
Judge mode: offline
```

Public subscore:

```text
correct  = 0.83
quality  = 0.8907
error    = 1.0
latency  = 0.625
cost     = 0.2749
drift    = 0.6674
prompt   = 0.9157
```

Public phase đã đạt điểm tối đa `100/100` sau khi cập nhật findings phù hợp với public scorer.

## 6. Private phase

Private v6 dùng test set kín và có thêm prompt injection twist. Trước khi chạy private, dùng findings có `prompt_injection`:

```bash
cp solution/findings_with_private_injection.json solution/findings.json
```

Chạy private simulator:

```bash
./bin/private/observathon-sim --config solution/config.json --wrapper solution/wrapper.py --out run_output.json --concurrency 8
```

Chấm điểm private:

```bash
./bin/private/observathon-score --run run_output.json --findings solution/findings.json --team local-team --out score.json
```

Kết quả private mới nhất:

```text
80/80 status ok
Private score: 92.22 / 100
Correct: 49 / 80
Diagnosis F1: 1.0
Judge mode: offline
```

Private subscore:

```text
correct  = 0.6725
quality  = 0.7988
error    = 1.0
latency  = 0.6259
cost     = 0.0
drift    = 0.782
prompt   = 0.8295
```

Private phase được dừng ở mốc `92.22/100` theo scorer mới nhất.

## 7. Các tối ưu đã thực hiện

### Config

`solution/config.json` được chỉnh theo hướng ổn định và giảm lỗi:

- Provider: `openai`
- Model: `gpt-5.4-nano`
- Temperature: `0.2`
- Bật `loop_guard`, `retry`, `cache`, `normalize_unicode`, `redact_pii`
- Giảm `context_size`, `max_steps`, `max_completion_tokens`
- Đặt `tool_budget = 4`
- Đặt `tool_error_rate = 0.0`
- Xóa `catalog_override` sai

### Prompt

`solution/prompt.txt` được viết ngắn gọn, tập trung vào:

- Gọi tool trước khi trả lời
- Tách riêng product, quantity, coupon và destination
- Chỉ dùng dữ liệu từ tool
- Tính subtotal, discount, shipping và total bằng số học nguyên
- Không lặp lại email hoặc số điện thoại của khách hàng
- Chống prompt injection trong ghi chú đơn hàng
- Trả lời tổng tiền theo dạng `Tong cong: <integer> VND`

### Wrapper

`solution/wrapper.py` là lớp giảm thiểu lỗi chính:

- Ghi observability qua telemetry logger
- Retry có kiểm soát khi gặp lỗi tạm thời
- Cache kết quả theo câu hỏi đã chuẩn hóa
- Redact PII trong câu trả lời
- Xóa note hoặc instruction không tin cậy
- Canonicalize câu hỏi thành format ngắn và rõ hơn
- Sửa mojibake địa danh như Hà Nội, Hải Phòng, Đà Nẵng, Đà Lạt
- Chuẩn hóa đáp án dựa trên trace tool
- Fallback khi tool trả lỗi tạm thời `loyalty_service_down`

### Findings

`solution/findings.json` và `solution/findings_with_private_injection.json` mô tả các fault class chính:

- `arithmetic_error`
- `fabrication`
- `tool_overuse`
- `infinite_loop`
- `tool_failure`
- `pii_leak`
- `error_spike`
- `quality_drift`
- `cost_blowup`
- `latency_spike`
- `prompt_injection`

## 8. Files nên push

Nên push:

```text
solution/config.json
solution/config_practice.json
solution/prompt.txt
solution/wrapper.py
solution/findings.json
solution/findings_with_private_injection.json
solution/examples.json
student_work/lab13_build.md
.gitignore
```

Không nên push:

```text
.env
bin/
logs/
run_output.json
score.json
debug_questions.json
debug_output.json
__pycache__/
*.pyc
```

## 9. Lệnh kiểm tra trước khi push

Chạy selfcheck:

```bash
python harness/selfcheck.py
```

Nếu dùng Python bundled trên Windows:

```powershell
C:\Users\nguye\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe harness\selfcheck.py
```

Kiểm tra Git:

```bash
git status
git add solution student_work/lab13_build.md .gitignore
git commit -m "Complete Day 13 Observathon lab"
git push origin main
```

## 10. Kết luận

Bài lab đã hoàn thành cả practice, public và private.

- Practice: `20/20 status ok`
- Public: `100.0/100`, `93/120 correct`, diagnosis F1 `1.0`
- Private: `92.22/100`, `49/80 correct`, diagnosis F1 `1.0`

Public phase đã đạt mức tối đa. Private phase dừng ở mốc `92.22/100` theo scorer mới nhất.