# Lab 13 Build Notes

## 1. Muc tieu lab

Day-13 Observathon yeu cau quan sat va toi uu mot e-commerce agent dang black-box. Agent co the tinh sai, goi tool sai, bi loi tool, lo PII, bi prompt injection va drift theo session. Bai lam tap trung vao 4 file chinh trong thu muc `solution/`:

- `config.json`
- `prompt.txt`
- `wrapper.py`
- `findings.json`

Ngoai ra co them `config_practice.json` de chay practice v5 mock engine.

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

Ket qua practice da dat:

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

Ket qua public da ghi nhan:

```text
120/120 status ok
Public score: 99.99 / 100
Correct: 92 / 120
Diagnosis F1: 0.952
```

Ghi chu: public score 99.99 da gan cham toi da. Khac biet nho chu yeu lien quan den diagnosis/findings truoc khi tach public/private.

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
```

Chi tiet diem private:

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
- Khong lap lai email/so dien thoai
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

## 8. Files can nop hoac nen push

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

Bai lab da hoan thanh ca practice, public va private. Public dat gan toi da `99.99/100`. Private dung scorer moi nhat dat `92.22/100`, voi `80/80` request chay thanh cong va diagnosis F1 dat `1.0`.