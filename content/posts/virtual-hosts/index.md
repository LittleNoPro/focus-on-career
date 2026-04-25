---
title: "How to set up Apache Virtual Hosts on Ubuntu server?"
date: 2025-09-02T23:30:00+07:00
draft: false
tags: ["Linux"]
categories: ["Configuration"]
---

## What is Virtual Hosts?

**Virtual Hosts** là một kĩ thuật cấu hình trên máy chủ web cho phép một máy chủ vật lý hoặc VPS duy nhất phục vụ nhiều website/domain khác nhau, giúp tiết kiệm chi phí và quản lý tập trung.

Chức năng chính của Virtual Hosts:
- **Phân biệt và định tuyến yêu cầu**: dựa vào domain name, IP, hoặc port trong request của client, web server sẽ xác định Virtual Host nào chịu trách nhiệm xử lí và trả về nội dung website tương ứng.
- **Quản lý tài nguyên**: cho phép chia sẻ tài nguyên của server (CPU, RAM, banwidth) một cách hiệu quả cho nhiều website.
- **Phục vụ nội dung độc lập**: mỗi Virtual Host có thể có thư mục chứa mã nguồn, file log và các thiết lập cấu hình riêng biệt.

Khi client gõ một tên miền vào trình duyệt, **HTTP Request** được gửi đến web server. Web server (ví dụ Apache hoặc Nginx) sẽ đọc thông tin trong **Host header** của HTTP request để xác định tên miền mà client muốn truy cập. Dựa vào tên miền này và các cấu hình Virtual Host đã được thiết lập, web server sẽ chuyển yêu cầu đến đúng thư mục chứa mã nguồn của website tương ứng. Cấu hình **DNS** đóng vai trò trỏ tên miền về địa chỉ IP của máy chủ.

## Set up Apache Virtual Hosts on Ubuntu

