# Lab Report — Ngày 26: Hạ Tầng MCP/A2A & Agentic Routing

- Tên: Nguyễn Ngọc Dũng<YOUR NAME></your>
- Ngày: 2026-07-02
- Repo: Day26-Track02-Cohort2-MCP-A2A-Infrastructure

## Mục tiêu

- Thiết lập môi trường và chạy hệ demo gồm một `orchestrator` (MCP) và ba A2A specialists (`search_agent`, `database_agent`, `synthesis_agent`).
- Thực hành routing giữa agents qua A2A, áp dụng governance guard, và ghi audit.
- Thu thập artifacts để nộp: agent-cards, logs, screenshots, notebook export và báo cáo.

## Môi trường đã dùng

- OS: Windows 10/11 (PowerShell)
- Python: quản lý bằng Conda (`pii-env`) — khuyến nghị Python 3.12
- Thư viện chính: `google-adk[a2a]`, `mcp`, `uvicorn`, `httpx`, `jupyter`

> Lưu ý: KHÔNG bao gồm file `.env` chứa `GOOGLE_API_KEY` trong nộp bài.

## Các tệp quan trọng trong repo

- `README.md` — hướng dẫn lab
- `agents/orchestrator/agent.py` — entrypoint orchestrator
- `agents/*_agent/agent.py` — entrypoint cho search/database/synthesis
- `scripts/start_a2a_servers.sh` & `scripts/start_adk_web.sh` — script tiện ích
- `day26_mcp_a2a_lab.ipynb` — notebook lab

## Bước thực hiện (từng bước, copy-paste)

1) Tạo và activate Conda environment

```powershell
conda create -n pii-env python=3.12 -y
conda activate pii-env
```

2) Cài dependencies và chuẩn bị môi trường

```powershell
pip install -r requirements.txt
copy .env.example .env
# set PYTHONPATH cho session PowerShell
$env:PYTHONPATH = (Get-Location).Path
```

3) Kiểm tra lệnh `adk` (trong cùng env)

```powershell
Get-Command adk
adk --version
# Nếu không có:
pip install "google-adk[a2a]"
$env:PATH = "$($env:CONDA_PREFIX)\Scripts;$env:PATH"
```

4) Khởi 3 A2A specialists (nếu chưa khởi): mở 3 cửa sổ PowerShell hoặc chạy nền

```powershell
python -m uvicorn agents.search_agent.agent:a2a_app --host localhost --port 8001
python -m uvicorn agents.database_agent.agent:a2a_app --host localhost --port 8002
python -m uvicorn agents.synthesis_agent.agent:a2a_app --host localhost --port 8003
```

(Thay thế bằng `Start-Process` nếu muốn chạy nền.)

5) Kiểm tra agent-card (lưu JSON)

```powershell
mkdir screenshots
curl http://localhost:8001/.well-known/agent-card.json -o screenshots/search_agent_card.json
curl http://localhost:8002/.well-known/agent-card.json -o screenshots/database_agent_card.json
curl http://localhost:8003/.well-known/agent-card.json -o screenshots/synthesis_agent_card.json
```

6) Khởi ADK Web cho `orchestrator`

```powershell
adk web agents/orchestrator
# Mở http://localhost:8000
```

7) Mở notebook và chạy cell demo

```powershell
jupyter notebook day26_mcp_a2a_lab.ipynb
```

Trong ADK Web hoặc notebook, thử các prompt sau để kiểm tra routing:

- Prompt 1 (Search): “Tìm tài liệu về data governance trong hệ multi-agent” — `orchestrator` nên ủy quyền `search_agent`.
- Prompt 2 (Database): “Cho tôi metrics hoạt động của các agent” — `orchestrator` nên ủy quyền `database_agent`.
- Prompt 3 (Synthesis): “Tổng hợp một executive summary từ findings sau” — `orchestrator` nên ủy quyền `synthesis_agent`.

Mỗi lần dispatch sẽ được ghi audit vào `logs/governance_audit.jsonl`.

## Kết quả mong đợi / Kiểm tra

- 3 agent-cards JSON tồn tại trong `screenshots/`.
- ADK Web UI hiển thị `orchestrator` và cho phép gửi prompt.
- `logs/governance_audit.jsonl` ghi các event gọi tool/dispatch.
- Notebook hiển thị outputs tương ứng với các tool calls.

## Thu thập artifacts (để nộp)

- `lab_report.md` (file này)
- `screenshots/` gồm: `adk_web.png`, `search_agent_card.json`, `database_agent_card.json`, `synthesis_agent_card.json`, `notebook_output.png`
- `logs/` (lấy subset: `governance_audit.jsonl`, `search_agent.log`, `database_agent.log`, `synthesis_agent.log`)
- `day26_mcp_a2a_lab.ipynb` (hoặc HTML export)

Lệnh hữu ích:

```powershell
# export notebook sang HTML
jupyter nbconvert --to html day26_mcp_a2a_lab.ipynb --output outputs/lab_notebook.html

# copy logs
mkdir outputs
Copy-Item logs\governance_audit.jsonl outputs\governance_audit.jsonl
Copy-Item logs\*.log outputs\ -Recurse

# tạo zip nộp (ví dụ)
Compress-Archive -Path lab_report.md, screenshots, outputs, day26_mcp_a2a_lab.ipynb -DestinationPath day26_submission.zip
```

## Troubleshooting (các lỗi thường gặp)

- WSL not installed error: nếu muốn chạy script bash, cài WSL hoặc dùng PowerShell theo hướng dẫn.
- `adk` command not found: chắc chưa cài `google-adk` trong env hoặc PATH chưa cập nhật — cài lại hoặc thêm `CONDA_PREFIX\Scripts` vào PATH.
- `uvicorn` lỗi import `google.adk`: đảm bảo pip install `-r requirements.txt` trong env đang active.
- Cổng bị chiếm: tìm PID và kill

```powershell
netstat -ano | findstr :8001
Stop-Process -Id <PID>
```

## Ghi chú bảo mật

- Không đính kèm `.env` hoặc bất kỳ file nào chứa `GOOGLE_API_KEY`.
- Nếu cần minh họa, replace key bằng `<REDACTED>`.

## Kết luận

- Lab mô phỏng một lớp nhỏ của hệ multi-agent với MCP + A2A và governance.
- Sau khi hoàn tất các bước trên, bạn có thể thử mở rộng prompt, thay đổi policy trong `lab_utils/governance/policy.json`, hoặc thêm tool mới vào specialist.

---

Nếu bạn muốn, tôi sẽ tiếp:

- Tự chạy các lệnh kiểm tra agent-card và lưu vào `screenshots/` (nếu tôi có quyền curl), hoặc
- Tạo hộ bạn thư mục `screenshots/` và một mẫu `adk_web.png` placeholder, hoặc
- Tự tạo `day26_submission.zip` (bạn cần cho phép tôi đọc logs và screenshot hiện có).

Hết.
