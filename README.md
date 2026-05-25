# DCT Blind Geo Forensics

Bài thực hành Labtainers khảo sát tấn công hình học vào kỹ thuật giấu tin trong ảnh bằng DCT (Discrete Cosine Transform), đồng thời mở rộng sang bài toán **blind geometric forensics**: phục hồi thông điệp khi không biết chính xác ảnh đã bị biến đổi hình học như thế nào.

Sinh viên sẽ trực tiếp quan sát ảnh hưởng của translation, crop, rotation, scaling và composite geometric attack tới BER, sau đó thực hiện resynchronization và blind recovery để khôi phục thông điệp đã nhúng.

---

# Mục tiêu

Sau khi hoàn thành bài lab, sinh viên có thể:

- Hiểu cơ chế giấu tin bằng DCT block 8x8.
- Hiểu vai trò của hệ số mid-frequency trong watermarking/steganography.
- Quan sát ảnh hưởng của geometric attack tới BER và khả năng khôi phục thông điệp.
- Phân tích nguyên nhân mất đồng bộ block DCT khi ảnh bị dịch, cắt, xoay hoặc resize.
- Thử nghiệm defense bằng geometric resynchronization.
- Thực hiện blind recovery bằng cách quét nhiều giả thuyết hình học.
- Đánh giá trade-off giữa robustness, PSNR, repeat factor, alpha và capacity.

---

# Kiến thức chính

## DCT

DCT biến đổi ảnh từ miền không gian sang miền tần số.

Trong bài lab, ảnh được xử lý theo các block 8x8. Thông điệp được nhúng vào cặp hệ số mid-frequency:

```text
(3,4) và (4,3)
```

Ý tưởng nhúng:

```text
bit = 1 nếu coeff(3,4) > coeff(4,3)
bit = 0 nếu coeff(3,4) < coeff(4,3)
```

Kỹ thuật này hạn chế méo ảnh hơn so với nhúng vào low-frequency, đồng thời bền vững hơn so với nhúng vào high-frequency.

---

## Geometric Attack

Các tấn công hình học làm thay đổi vị trí pixel hoặc cấu trúc không gian của ảnh, từ đó phá vỡ ranh giới block 8x8 ban đầu.

Các attack trong bài lab gồm:

- Translation
- Crop
- Rotation
- Scaling
- Composite attack

Khi block extraction không còn trùng với block đã dùng để nhúng, BER sẽ tăng mạnh và thông điệp khôi phục có thể bị sai.

---

## Blind Geometric Forensics

Blind forensics trong bài lab này nghĩa là sinh viên không được biết chính xác tham số attack ban đầu.

Thay vì chỉ extract trực tiếp, sinh viên phải thử nhiều giả thuyết hình học như:

- crop percent
- rotation angle
- translation dx,dy

Mục tiêu là tìm phép bù hình học đủ tốt để đồng bộ lại block DCT và làm BER giảm xuống dưới ngưỡng yêu cầu.

Thông điệp cần khôi phục:

```text
DCT BLIND GEO FORENSIC B22DCAT164
```

---

# Tải bài lab

```bash
imodule https://github.com/KhoaPTIT/dct-blind-geo-forensics/raw/main/dct-blind-geo-forensics.tar
```

---

# Khởi động bài lab

```bash
labtainer dct-blind-geo-forensics
```

Nếu cần rebuild khi đang phát triển hoặc sửa lab:

```bash
rebuild dct-blind-geo-forensics
```

---

# Nội dung thực hành

## Task 1 — Generate Cover Image và DCT Embedding

Sinh viên tạo ảnh cover, sau đó nhúng thông điệp vào ảnh bằng DCT watermarking.

Tạo ảnh cover:

```bash
python3 generate_cover.py
```

Nhúng thông điệp:

```bash
python3 embed_message.py --alpha 35 --repeat 5
```

Mở ảnh để quan sát:

```bash
display cover.png
display stego.png
```

Sinh viên cần quan sát:

- ảnh `cover.png` là ảnh gốc được tạo tự động.
- ảnh `stego.png` là ảnh đã nhúng thông điệp.
- giá trị PSNR hiển thị trên terminal.
- ảnh stego nhìn gần giống ảnh cover nếu quá trình nhúng không gây méo lớn.

Kết quả checkwork liên quan:

```text
Y - embed
```

---

## Task 2 — Clean Extraction

Sinh viên trích xuất thông điệp từ ảnh stego khi chưa có attack để kiểm tra baseline.

Chạy:

```bash
python3 extract_message.py --image stego.png --repeat 5
```

Sinh viên cần quan sát:

- thông điệp khôi phục trên terminal.
- BER baseline.
- nếu nhúng và trích xuất đúng, BER phải bằng 0%.

Kết quả mong đợi:

```text
Clean extraction BER = 0.00%
```

Kết quả checkwork liên quan:

```text
Y - extract
```

---

## Task 3 — DCT Visualization

Sinh viên trực quan hóa phổ DCT để quan sát phân bố năng lượng trong miền tần số.

Chạy:

```bash
python3 visualize_dct.py
```

Mở ảnh phổ DCT:

```bash
display dct_spectrum_before.png
```

Sinh viên cần quan sát:

- năng lượng tập trung nhiều ở vùng low-frequency.
- vùng mid-frequency được dùng để nhúng thông điệp.
- high-frequency thường dễ bị suy giảm sau nén hoặc nội suy ảnh.

Kết luận cần rút ra:

- low-frequency bền hơn nhưng dễ gây méo ảnh.
- high-frequency ít ảnh hưởng thị giác nhưng dễ mất dữ liệu.
- mid-frequency là vùng cân bằng giữa imperceptibility và robustness.

Kết quả checkwork liên quan:

```text
Y - dct_visual
```

---

## Task 4 — Translation Attack

Sinh viên thực hiện translation attack để khảo sát ảnh hưởng của dịch chuyển ảnh tới BER.

Chạy lần lượt:

```bash
python3 attack_translation.py --dx 2 --dy 2
python3 attack_translation.py --dx 5 --dy 5
python3 attack_translation.py --dx 10 --dy 10
python3 attack_translation.py --dx 20 --dy 20
```

Mở ảnh attacked:

```bash
display attacks/translation_20_20.png
```

Sinh viên cần quan sát:

- ảnh bị dịch khỏi vị trí ban đầu.
- vùng biên xuất hiện padding.
- BER thay đổi khi mức dịch chuyển tăng.
- extractor đọc sai block DCT do mất đồng bộ block 8x8.

Kết luận cần rút ra:

- translation attack không nhất thiết làm ảnh biến dạng mạnh về mặt thị giác.
- tuy nhiên chỉ cần lệch vài pixel cũng có thể làm sai block alignment.
- DCT watermarking phụ thuộc mạnh vào vị trí block ban đầu.

Kết quả checkwork liên quan:

```text
Y - translate
```

---

## Task 5 — Crop Attack

Sinh viên khảo sát ảnh hưởng của crop attack tới khả năng khôi phục thông điệp.

Chạy lần lượt:

```bash
python3 attack_crop.py --percent 5
python3 attack_crop.py --percent 10
python3 attack_crop.py --percent 20
python3 attack_crop.py --percent 30
```

Mở ảnh attacked:

```bash
display attacks/crop_30.png
```

Sinh viên cần quan sát:

- ảnh bị mất vùng biên.
- ảnh sau crop được resize lại về kích thước ban đầu.
- resize làm thay đổi nội dung pixel và ranh giới block.
- BER thường tăng khi crop mạnh hơn.

Kết luận cần rút ra:

- crop attack vừa làm mất dữ liệu watermark ở vùng bị cắt.
- vừa gây sai lệch block extraction do quá trình resize.
- crop càng lớn thì khả năng phục hồi thông điệp càng giảm.

Kết quả checkwork liên quan:

```text
Y - crop
```

---

## Task 6 — Rotation Attack

Sinh viên thực hiện rotation attack để khảo sát mức độ phá hủy đồng bộ của phép xoay ảnh.

Chạy lần lượt:

```bash
python3 attack_rotation.py --angle 1
python3 attack_rotation.py --angle 3
python3 attack_rotation.py --angle 5
python3 attack_rotation.py --angle 10
```

Mở ảnh attacked:

```bash
display attacks/rotation_10.png
```

Sinh viên cần quan sát:

- ảnh bị xoay quanh tâm.
- xuất hiện vùng padding ở góc ảnh.
- phép xoay tạo sai lệch không đều trên toàn ảnh.
- BER có thể tăng nhanh dù góc xoay nhỏ.

Kết luận cần rút ra:

- rotation attack là một trong các geometric attack nguy hiểm nhất với DCT watermarking.
- quá trình nội suy sau rotation làm thay đổi hệ số DCT.
- rotation phá đồng bộ mạnh hơn translation đơn giản.

Kết quả checkwork liên quan:

```text
Y - rotate
```

---

## Task 7 — Scaling Attack

Sinh viên khảo sát ảnh hưởng của scaling attack tới khả năng khôi phục thông điệp.

Chạy lần lượt:

```bash
python3 attack_scaling.py --scale 90
python3 attack_scaling.py --scale 75
python3 attack_scaling.py --scale 50
```

Mở ảnh attacked:

```bash
display attacks/scaling_75.png
```

Sinh viên cần quan sát:

- ảnh bị resize xuống rồi resize lại về kích thước ban đầu.
- nội suy làm thay đổi giá trị pixel.
- hệ số DCT thay đổi sau scaling.
- BER tăng khi scale nhỏ hơn.

Kết luận cần rút ra:

- scaling attack gây biến đổi cả miền không gian và miền tần số.
- watermark bị suy giảm do quá trình nội suy ảnh.
- scale càng thấp thì thông điệp càng khó phục hồi.

Kết quả checkwork liên quan:

```text
Y - scale
```

---

## Task 8 — Composite Geometric Attack

Sinh viên thực hiện composite attack bằng cách kết hợp nhiều phép biến đổi hình học cùng lúc.

Chạy:

```bash
python3 attack_composite.py --angle 3 --dx 5 --dy 5 --crop 10
```

Mở ảnh attacked:

```bash
display attacks/composite_attacked.png
```

Sinh viên cần quan sát:

- ảnh bị xoay, dịch chuyển và crop đồng thời.
- BER thường cao hơn nhiều attack đơn lẻ.
- block alignment bị phá hủy nghiêm trọng.
- extractor khó khôi phục đúng thông điệp nếu không có defense.

Kết luận cần rút ra:

- composite attack nguy hiểm hơn geometric attack riêng lẻ.
- nhiều biến đổi hình học cộng dồn làm quá trình đồng bộ lại block DCT khó hơn.

Kết quả checkwork liên quan:

```text
Y - composite
```

---

## Task 9 — Resynchronization Defense

Sinh viên thử nghiệm defense bằng geometric resynchronization để giảm BER sau composite attack.

Chạy defense:

```bash
python3 registration_defense.py
```

Nếu defense chưa pass, mở file để chỉnh tham số:

```bash
nano registration_defense.py
```

Các tham số quan trọng:

```python
SEARCH_RADIUS = 8
ROTATION_CANDIDATES = [-5, -3, -1, 0, 1, 3, 5]
SCALE_CANDIDATES = [0.90, 0.95, 1.0, 1.05, 1.10]
REPEAT_N = 5
ALPHA = 35.0
```

Sau khi chỉnh sửa, chạy lại:

```bash
python3 registration_defense.py
```

Mở ảnh đã defense:

```bash
display defended.png
```

Sinh viên cần quan sát:

- defense thử nhiều cấu hình scale, rotation và translation.
- script chọn cấu hình có BER tốt nhất.
- defense không cần khôi phục ảnh hoàn hảo, chỉ cần đồng bộ đủ tốt để đọc lại DCT block.

Điều kiện pass chính:

```text
Defense BER <= 15%
PSNR >= 35 dB
SEARCH_RADIUS >= 8
REPEAT_N >= 5
25 <= ALPHA <= 60
```

Kết quả checkwork liên quan:

```text
Y - defense
```

---

## Task 10 — Blind Challenge Generation

Sinh viên tạo ảnh challenge bị tấn công hình học ẩn tham số.

Chạy:

```bash
python3 challenge_generate.py
```

Mở ảnh challenge:

```bash
display challenge_blind.png
```

Sinh viên cần quan sát:

- `challenge_blind.png` đã bị biến đổi hình học kết hợp.
- direct extraction thường cho BER cao.
- không nên chỉ extract trực tiếp vì block DCT đã bị mất đồng bộ.

Kết quả checkwork liên quan:

```text
Y - blind_make
```

---

## Task 11 — Blind Recovery

Sinh viên phục hồi thông điệp từ ảnh blind challenge bằng cách quét nhiều giả thuyết crop, rotation và translation.

Chạy mặc định:

```bash
python3 recover_challenge.py
```

Nếu chưa pass, mở rộng không gian tìm kiếm:

```bash
python3 recover_challenge.py --radius 10 --crops 6,8,10,12,14,16 --angles -8,-6,-5,-4,-3,-2,0,2,3,4,5,6,8
```

Mở ảnh đã khôi phục:

```bash
display challenge_recovered.png
```

Sinh viên cần quan sát:

- script thử nhiều candidate khác nhau.
- mỗi candidate là một giả thuyết bù crop, rotation, translation.
- candidate tốt nhất được chọn theo BER thấp nhất.
- thông điệp khôi phục càng gần chuỗi gốc thì BER càng thấp.

Điều kiện pass chính:

```text
Best BER <= 12%
Candidates tested >= 200
challenge_recovery_report.txt có dòng: Blind geometric DCT challenge recovered
```

Kết quả checkwork liên quan:

```text
Y - blind_recover
```

---

## Task 12 — Summary Report

Sinh viên tạo báo cáo tổng hợp BER sau khi đã chạy đầy đủ các bước trên.

Chạy:

```bash
python3 summary.py
```

Xem báo cáo:

```bash
cat summary_report.txt
```

Sinh viên cần tổng hợp:

- BER baseline.
- BER sau từng geometric attack.
- BER sau defense.
- BER tốt nhất trong blind challenge.
- nhận xét attack nào làm hệ thống suy giảm mạnh nhất.

Kết quả checkwork liên quan:

```text
Y - summary
```

---

# Checkwork

Sau khi hoàn thành toàn bộ task, chạy:

```bash
checkwork
```

Kết quả mong đợi:

```text
Y - embed
Y - extract
Y - dct_visual
Y - translate
Y - crop
Y - rotate
Y - scale
Y - composite
Y - defense
Y - blind_make
Y - blind_recover
Y - summary
```

Nếu một mục vẫn là `N`, hãy kiểm tra report tương ứng:

```bash
ls -lh *.txt
cat embed_report.txt
cat baseline_report.txt
cat visual_report.txt
cat translation_report.txt
cat crop_report.txt
cat rotation_report.txt
cat scaling_report.txt
cat composite_report.txt
cat defense_report.txt
cat challenge_report.txt
cat challenge_recovery_report.txt
cat summary_report.txt
```

---

# Bộ lệnh chạy nhanh

Có thể chạy toàn bộ lab theo thứ tự sau:

```bash
python3 generate_cover.py
python3 embed_message.py --alpha 35 --repeat 5
python3 extract_message.py --image stego.png --repeat 5
python3 visualize_dct.py

python3 attack_translation.py --dx 2 --dy 2
python3 attack_translation.py --dx 5 --dy 5
python3 attack_translation.py --dx 10 --dy 10
python3 attack_translation.py --dx 20 --dy 20

python3 attack_crop.py --percent 5
python3 attack_crop.py --percent 10
python3 attack_crop.py --percent 20
python3 attack_crop.py --percent 30

python3 attack_rotation.py --angle 1
python3 attack_rotation.py --angle 3
python3 attack_rotation.py --angle 5
python3 attack_rotation.py --angle 10

python3 attack_scaling.py --scale 90
python3 attack_scaling.py --scale 75
python3 attack_scaling.py --scale 50

python3 attack_composite.py --angle 3 --dx 5 --dy 5 --crop 10
python3 registration_defense.py
python3 challenge_generate.py
python3 recover_challenge.py
python3 summary.py
checkwork
```

---

# Kết luận mong đợi

Sau bài lab, sinh viên cần rút ra:

- DCT watermarking phụ thuộc mạnh vào block alignment.
- Geometric attack làm tăng BER do mất đồng bộ block 8x8.
- Translation nhỏ vẫn có thể gây lỗi lớn vì extractor đọc sai ranh giới block.
- Crop và scaling làm thay đổi dữ liệu watermark do resize và nội suy.
- Rotation và composite attack thường nguy hiểm nhất vì tạo sai lệch hình học phức tạp.
- Tăng `alpha` và `repeat` có thể cải thiện robustness nhưng làm giảm imperceptibility hoặc capacity.
- Resynchronization defense giúp giảm BER nhưng không đảm bảo khôi phục hoàn hảo.
- Blind recovery cần thử nhiều giả thuyết hình học để tìm cấu hình bù tốt nhất.

---

# Thông tin học phần

Hoàng Anh Khoa — B22DCAT164

Lớp: D22CQAT04-B

Học phần: Kỹ thuật giấu tin (INT14102)

Học viện Công nghệ Bưu chính Viễn thông (PTIT)

Giảng viên hướng dẫn: PGS.TS. Đỗ Xuân Chợ
