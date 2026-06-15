# Lab 13 Build Notes

## Muc tieu
Lab Day-13 Observathon yeu cau quan sat mot agent thuong mai dien tu dang hop den, chan doan cac loi van hanh va toi uu ket qua bang `config.json`, `prompt.txt`, `wrapper.py` va `findings.json`.

## Moi truong va binary
- Chay tren WSL/Ubuntu vi binary Windows bi loi PyInstaller `python312.dll` tren may hien tai.
- Practice v5 dung mock engine, khong can API key: binary dat tai `bin/practice/observathon-sim`.
- Public v6 dung real LLM/OpenAI: binary dat tai `bin/public/observathon-sim` va scorer `bin/public/observathon-score`.
- Private v6 dung test set kin va injection twist: binary dat tai `bin/private/observathon-sim` va scorer `bin/private/observathon-score`.
- API key duoc luu trong `.env`; file nay khong duoc commit.

## Huong toi uu da thuc hien
- `solution/config.json`: cau hinh OpenAI `gpt-5.4-nano`, temperature thap, bat retry/cache/normalize/redact, gioi han steps, token va tool budget.
- `solution/config_practice.json`: cau hinh mock rieng cho practice v5.
- `solution/prompt.txt`: prompt ngan, yeu cau tool-first, dung du lieu tool, tinh so hoc chinh xac, khong lo PII va chong prompt injection.
- `solution/wrapper.py`: them observability, cache, retry co kiem soat, sanitize note/instruction, canonicalize cau hoi, sua mojibake dia danh, chuan hoa output va fallback khi tool loi tam thoi `loyalty_service_down`.
- `solution/findings.json`: findings hien tai dung cho private, co 11 fault classes va dat diagnosis F1 1.0.
- `solution/findings_with_private_injection.json`: ban backup private co `prompt_injection`.

## Lenh chay chinh
Nap API key trong WSL:

```bash
cd /mnt/d/AI20k/lab13/Day-13-Lab-Observathon
export OPENAI_API_KEY="$(grep '^OPENAI_API_KEY=' .env | cut -d= -f2- | tr -d '\r' | sed 's/^"//;s/"$//')"
```

Practice v5:

```bash
./bin/practice/observathon-sim --config solution/config_practice.json --wrapper solution/wrapper.py --out run_output.json
```

Public v6:

```bash
./bin/public/observathon-sim --config solution/config.json --wrapper solution/wrapper.py --out run_output.json --concurrency 8
./bin/public/observathon-score --run run_output.json --findings solution/findings.json --team local-team --out score.json
```

Private v6:

```bash
cp solution/findings_with_private_injection.json solution/findings.json
./bin/private/observathon-sim --config solution/config.json --wrapper solution/wrapper.py --out run_output.json --concurrency 8
./bin/private/observathon-score --run run_output.json --findings solution/findings.json --team local-team --out score.json
```

## Ket qua
- Practice v5: `20/20 status ok` voi mock engine.
- Public v6: `120/120 status ok`; score truoc do dat `99.99/100`, `92/120` correct, diagnosis F1 khoang `0.952` khi findings chua tach public/private.
- Private v6 moi nhat: `80/80 status ok`; score `92.22/100`, `49/80` correct, diagnosis F1 `1.0`.

Private score moi nhat:

```text
HEADLINE: 92.22 / 100
correct: 0.6725
quality: 0.7988
error: 1.0
latency: 0.6259
cost: 0.0
drift: 0.782
prompt: 0.8295
diagnosis F1: 1.0
```

## Luu y khi push
Khong commit cac file chua secret, binary hoac output tam:

```text
.env
bin/
logs/
run_output.json
score.json
debug_questions.json
debug_output.json
```

Nen push cac file can nop trong `solution/` va file ghi chu nay. Khong hardcode question id, gia tri dap an hoac doc file instructor/scorer noi bo; cac thay doi hien tai dua tren prompt, config, wrapper guardrail va telemetry hop le.