---
title: "Volatility 3: The Next Generation of Memory Forensics"
date: 2026-02-28T23:30:00+07:00
draft: false
tags: ["Tools"]
categories: ["Linux", "Window"]
---

## Introduction 

**Volatility 3** là bước phát triển của một trong những công cụ mã nguồn mở mạnh nhất trong lĩnh vực **Digital Forensics** - một framework viết bằng **Python 3** chuyên phân tích các **memory dump** từ hệ thống Windows, Linux và macOS. Cốt lõi của **Volatility 3** là trích xuất các **artifact** số quan trọng như tiến trình đang chạy, kết nối mạng và thông tin xác thực của user từ các mẫu **RAM**. Những artifact này nổi tiếng là tồn tại rất ngắn ngủi, nhưng lại thường chứa bằng chứng giá trị nhất trong **Incident Response** và các cuộc điều tra **malware**.

Khác với các phiên bản trước đó, **Volatility 3** có kiến trúc **module**, phân lớp gồm **memory layers** (quản lý không gian địa chỉ), **symbol tables** (các cấu trúc đặc thù của từng OS), và **object templates** (dùng để phân tích cú pháp dữ liệu). Thiết kế này giúp cải thiện performance, hỗ trợ phân tích đa nền tảng và giúp việc phát triển plug-in tùy chỉnh dễ dàng hơn. Dù là tái dựng trạng thái hệ thống hay điều tra các mối đe dọa APT. **Volatility 3** giúp các analyzer nhìn thẳng vào vấn chính của một máy khi bị xâm nhập. 

## Understanding Plugins in Volatility 3

Các plugins là xương sống của **Volatility 3**. Chúng là các module chuyên biệt thực hiện những phân tích có mục tiêu trên **memory image**. Được tổ chức theo từng hệ điều hành, các plugin này tận dụng các thành phần cốt lõi của framework để phân tích cú pháp các cấu trúc hệ thống như **process lists** (danh sách tiến trình), **registry hives** và **network sockets**, từ đó tạo ra thông tin có thể hành động ngay (actionable intelligence) ở dạng dữ liệu có cấu trúc.

Điểm làm **Volatility 3** khác biệt là khả năng mở rộng plugin (plugin extensibility). Nhà phân tích có thể viết plugin tùy chỉnh bằng cách kế thừa từ *Plugin Interface* và định nghĩa các yêu cầu thông qua một *Hierarchical Dictionary*, cho phép thích ứng nhanh với các mối đe dọa mới mà không cần sửa đổi công cụ lõi.

Sau đây mình sẽ giới thiệu về những plugins phổ biến:

### pslist

Plugin **windows.pslist** liệt kê các **process** đang hoạt động bằng cách duyệt qua danh sách liên kết tiến trình của **kernel**, cung cấp các yếu tố thiết yếu như **PID, PPID, image name, ...**

```bash=
vol -f memdump.mem windows.pslist
```
### psscan

Khác với **pslist**, **windows.psscan** chủ động quét các **memory pools** để tìm cấu trúc **EPROCESS**, phát hiện các tiến trình ẩn, đã kết thúc hoặc bị inject mà việc duyệt **linked-list** có thể bỏ sót. Đây là plugin quan trọng để phát hiện **rootkit**.

```bash= 
vol -f memdump.mem windows.psscan
```

### getsids

**windows.getsids** khôi phục các **Security Identifier (SID)** gắn với một tiến trình, bao gồm nhóm chính và nhóm bổ sung. Plugin này giúp ánh xạ ngữ cảnh người dùng và phát hiện **privilege escalation** trong các cuộc tấn công nhắm vào đặc quyền.

```bash=
vol -f memdump.mem windows.getsids
```

### privs 

**windows.privs** kiểm tra các đặc quyền của token trong một tiến trình, cho thấy những quyền đang được bật như debugging hoặc truy cập hệ thống (system access). Đây có thể là dấu hiệu của khai thác lỗ hổng hoặc nâng quyền trái phép.

```bash=
vol -f memdump.mem windows.privilege.Privs
```

### handles 

Plugin **windows.handles** liệt kê các handle đang mở (file, registry key) của một tiến trình, giúp lộ ra những tương tác đáng ngờ như khóa file bất thường hoặc mutex được malware dùng để giao tiếp.

```bash=
vol -f memdump.mem windows.handles
```

### cmdline

**windows.cmdline** tái dựng đầy đủ các tham số dòng lệnh (command-line arguments) được truyền cho tiến trình, thường làm lộ các tham số bị làm rối (obfuscated) hoặc vector injection trong script và file thực thi.

```bash=
vol -f memdump.mem windows.cmdline
```

### dlllist

Plugin **windows.dlllist** liệt kê các DLL đã được nạp của từng tiến trình, gồm địa chỉ cơ sở (base address) và đường dẫn, giúp phát hiện code injection hoặc các thư viện không có chữ ký (unsigned), vốn là dấu hiệu của việc bị xâm nhập.

```bash=
vol -f memdump.mem windows.dlllist
```

### memmap

**windows.memmap.Memmap** trích xuất không gian địa chỉ ảo (virtual address space) của một tiến trình ra đĩa, cho phép kiểm tra sâu hơn các artifact trong heap hoặc stack.

```bash=
vol -f memdump.mem -o output_dir windows.memmap.Memmap --dump --pid 104     
```

### procdump

**windows.dumpfiles** trích xuất file thực thi của tiến trình (cùng các DLL liên quan) từ bộ nhớ, hữu ích để dịch ngược (reverse-engineering) các file nhị phân malware mà không cần truy cập ổ đĩa.

```bash= 
vol -f memdump.mem -o output_dir windows.dumpfiles --pid 104         
```

### PrintKey

Plugin **windows.registry.printkey.PrintKey** lấy ra các chính sách kiểm toán (audit policy) của hệ thống, giúp làm rõ các cấu hình ghi log có thể đã bị can thiệp để né tránh phát hiện (evade detection).

```bash=
vol -f memdump.mem windows.registry.printkey.PrintKey     
```

### hashdump

**windows.hashdump** trích xuất các password hash LM/NTLM từ SAM registry hive, cho phép bẻ khóa offline (offline cracking) để khôi phục thông tin xác thực, phục vụ phân tích di chuyển ngang (lateral movement).

```bash=
vol -f memdump.mem windows.hashdump.Hashdump
```

### hivelist

**windows.registry.hivelist** liệt kê các registry hive đang được nạp cùng địa chỉ ảo (virtual address) của chúng, là nền tảng cho các bước registry forensics tiếp theo như liệt kê key.

```bash=
vol -f memdump.mem windows.registry.hivelist.HiveList
```

### netscan

**windows.netscan** quét các kết nối TCP/UDP, cổng đang lắng nghe (listening ports) và các socket artifact, tái dựng hoạt động mạng để lần theo liên lạc C2 (command and control) hoặc hành vi exfiltration (rò rỉ dữ liệu).

```bash=
vol -f memdump.mem windows.netscan
```


