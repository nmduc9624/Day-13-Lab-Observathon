# Lab 13 Build Notes

## 1. Muc tieu lab

Day-13 Observathon yeu cau quan sat va toi uu mot e-commerce agent dang black-box. Agent co the tinh sai, goi tool sai, bi loi tool, lo PII, bi prompt injection va drift theo session. Bai lam tap trung vao cac file trong thu muc `solution/`:

- `config.json`
- `config_practice.json`
- `prompt.txt`
- `wrapper.py`
- `findings.json`
- `findings_with_private_injection.json`
- `examples.json`

## 2. Moi truong da dung

May Windows gap loi khi chay binary Windows:

```text
Failed to load Python DLL python312.dll
LoadLibrary: Invalid access to memory location
```

Vi vay lab duoc chay bang WSL Ubuntu voi cac binary Linux x64.

Cac binary da dung:

```text
bin/practice/observathon-sim
bin/public/observathon-sim
bin/public/observathon-score
bin/private/observathon-sim
bin/private/observathon-score
```

API key OpenAI duoc luu trong `.env`. File `.env` khong duoc commit len git.

## 3. Cach nap API key trong WSL

Chay trong thu muc lab:

```bash
cd /mnt/d/AI20k/lab13/Day-13-Lab-Observathon
export OPENAI_API_KEY="$(grep '^OPENAI_API_KEY=' .env | cut -d= -f2- | tr -d '\r' | sed 's/^"//;s/"$//')"
```

## 4. Practice phase

Practice v5 la mock engine, khong can API key. Dung file cau hinh rieng:

```bash
./bin/practice/observathon-sim --config solution/config_practice.json --wrapper solution/wrapper.py --out run_output.json
```

Ket qua practice:

```text
20/20 status ok
```

## 5. Public phase

Public v6 dung real LLM va OpenAI key.

Chay simulator:

```bash
./bin/public/observathon-sim --config solution/config.json --wrapper solution/wrapper.py --out run_output.json --concurrency 8
```

Cham diem:

```bash
./bin/public/observathon-score --run run_output.json --findings solution/findings.json --team local-team --out score.json
```

Ket qua public moi nhat:

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

Ghi chu: public score da dat toi da 100/100 sau khi cap nhat findings phu hop public.

## 6. Private phase

Private v6 co test set kin va them prompt injection twist. Truoc khi chay private, dung findings co `prompt_injection`:

```bash
cp solution/findings_with_private_injection.json solution/findings.json
```

Chay simulator:

```bash
./bin/private/observathon-sim --config solution/config.json --wrapper solution/wrapper.py --out run_output.json --concurrency 8
```

Cham diem:

```bash
./bin/private/observathon-score --run run_output.json --findings solution/findings.json --team local-team --out score.json
```

Ket qua private moi nhat:

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

## 7. Cac toi uu da lam

### Config

`solution/config.json` duoc chinh theo huong on dinh va tiet kiem:

- Provider: `openai`
- Model: `gpt-5.4-nano`
- Temperature: `0.2`
- Bat `loop_guard`, `retry`, `cache`, `normalize_unicode`, `redact_pii`
- Giam `context_size`, `max_steps`, `max_completion_tokens`
- Dat `tool_budget = 4`
- Dat `tool_error_rate = 0.0`
- Xoa `catalog_override` sai

### Prompt

`solution/prompt.txt` duoc viet ngan gon, tap trung vao:

- Goi tool truoc khi tra loi
- Tach product, quantity, coupon, destination
- Dung duy nhat du lieu tu tool
- Tinh subtotal, discount, shipping va total bang so hoc nguyen
- Khong lap lai email hoac so dien thoai
- Chong prompt injection trong ghi chu don hang
- Tra loi hop le theo dang `Tong cong: <integer> VND`

### Wrapper

`solution/wrapper.py` la lop giam thieu loi chinh:

- Ghi observability qua telemetry logger
- Retry co kiem soat khi gap loi tam thoi
- Cache ket qua theo cau hoi da chuan hoa
- Redact PII trong answer
- Xoa note/instruction khong tin cay
- Canonicalize cau hoi thanh format ngan, ro hon
- Sua mojibake dia danh nhu Ha Noi, Hai Phong, Da Nang, Da Lat
- Chuan hoa dap an dua tren trace tool
- Fallback khi tool tra loi `loyalty_service_down`

### Findings

`solution/findings.json` va `solution/findings_with_private_injection.json` mo ta cac fault class chinh:

- arithmetic_error
- fabrication
- tool_overuse
- infinite_loop
- tool_failure
- pii_leak
- error_spike
- quality_drift
- cost_blowup
- latency_spike
- prompt_injection

## 8. Files nen push

Nen push:

```text
solution/config.json
solution/config_practice.json
solution/prompt.txt
solution/wrapper.py
solution/findings.json
solution/findings_with_private_injection.json
solution/examples.json
lab13_build.md
.gitignore
```

Khong nen push:

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

## 9. Lenh kiem tra truoc khi push

Chay selfcheck:

```bash
python harness/selfcheck.py
```

Neu dung Python bundled tren Windows:

```powershell
C:\Users\nguye\.cache\codex-runtimes\codex-primary-runtime\dependencies\python\python.exe harness\selfcheck.py
```

Kiem tra git:

```bash
git status
git add solution lab13_build.md .gitignore
git commit -m "Complete Day 13 Observathon lab"
git push origin main
```

## 10. Ket luan

Bai lab da hoan thanh ca practice, public va private.

- Practice: 20/20 status ok
- Public: 100.0/100, 93/120 correct, diagnosis F1 1.0
- Private: 92.22/100, 49/80 correct, diagnosis F1 1.0

Public phase da dat muc toi da. Private phase da dung o moc 92.22/100 theo scorer moi nhat.