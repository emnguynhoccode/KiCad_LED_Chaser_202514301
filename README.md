# Dự án Thiết kế Mạch LED Chaser Sử Dụng IC NE555 & CD4017 (KiCad 10.0)

## Thông tin thực hiện
- **Sinh viên thực hiện:** Nguyễn Thanh Nguyên
- **MSSV:** `202514301`
- **Lớp:** ĐTVT 10 - K70 (Đại học Bách Khoa Hà Nội - HUST)

---

## Giới thiệu & Nguyên lý hoạt động
Mạch LED Chaser (LED chạy đuổi 10 kênh) được thiết kế hoàn chỉnh trên phần mềm **KiCad 10.0** sử dụng các linh kiện xuyên lỗ (THT/PTH) phổ biến, gồm 3 khối chức năng chính:

1. **Khối nguồn (`J1`):** Header 2 chân cấp nguồn một chiều `+9V` và `GND` cho toàn mạch.
2. **Khối tạo xung nhịp vuông (`U1 - NE555P`):**
   - Hoạt động ở chế độ dao động đa hài phi ổn (**Astable Multivibrator**), sử dụng điện trở `R1` (`10k`), biến trở tinh chỉnh tốc độ `RV1` (`10k`) và tụ định thời `C1` (`10uF`); tụ `C2` (`0.01uF`) lọc nhiễu tại chân điều khiển điện áp (`CONT`).
   - Khi xoay biến trở `RV1`, chu kỳ nạp/xả của tụ `C1` thay đổi, giúp điều chỉnh trực tiếp tần số xung vuông ngõ ra tại chân 3 (`OUT`).
   - Nhánh `D1` (`LED_CLK`) nối tiếp điện trở hạn dòng `R3` (`680R`) đóng vai trò đèn báo nhịp xung Clock trực tiếp từ `NE555P`.
3. **Khối đếm thập phân & Hiển thị (`U2 - CD4017`):**
   - IC đếm vòng Johnson 10 ngõ ra nhận xung Clock tại chân 14 (`CLK`), chân `CKEN` (13) và `Reset` (15) nối `GND` để đếm lặp liên tục từ `Q0` đến `Q9`.
   - Tại mỗi chu kỳ xung nhịp, một ngõ ra từ `Q0` đến `Q9` lên mức cao, kích sáng tuần tự 10 LED chạy đuổi `D2` – `D11`.
   - Toàn bộ các chân Cathode của `D2` – `D11` được mắc chung qua điện trở hạn dòng `R2` (`680R`) xuống `GND` để giới hạn dòng làm việc an toàn cho LED và IC.

---

## Bảng danh sách linh kiện (BOM)
Dữ liệu trích xuất đồng bộ từ [`output/bom/LED_chaser_BOM.csv`](output/bom/LED_chaser_BOM.csv) (tổng cộng **20 linh kiện**):

| STT | Reference | Value | Footprint | Số lượng |
| :-: | :--- | :--- | :--- | :-: |
| 1 | `C1` | `10uF` | `LED_Chaser_Fp:CP_Radial_D5.0mm_P2.50mm` | 1 |
| 2 | `C2` | `0.01uF` | `LED_Chaser_Fp:C_Disc_D5.0mm_W2.5mm_P2.50mm` | 1 |
| 3 | `D1` | `LED_CLK` | `LED_Chaser_Fp:LED_D5.0mm` | 1 |
| 4 | `D2 - D11` | `LED` | `LED_Chaser_Fp:LED_D5.0mm` | 10 |
| 5 | `J1` | `9V_IN` | `LED_Chaser_Fp:PinHeader_1x02_P2.54mm_Vertical` | 1 |
| 6 | `R1` | `10k` | `LED_Chaser_Fp:R_Axial_DIN0207_L6.3mm_D2.5mm_P10.16mm_Horizontal` | 1 |
| 7 | `R2, R3` | `680R` | `LED_Chaser_Fp:R_Axial_DIN0207_L6.3mm_D2.5mm_P10.16mm_Horizontal` | 2 |
| 8 | `RV1` | `10k` | `LED_Chaser_Fp:Potentiometer_Bourns_3386P_Vertical` | 1 |
| 9 | `U1` | `NE555P` | `LED_Chaser_Fp:DIP-8_W7.62mm` | 1 |
| 10 | `U2` | `CD4017` | `LED_Chaser_Fp:DIP-16_W7.62mm` | 1 |
| **Tổng** | | | | **20** |

---

## Cấu trúc thư mục dự án

```text
KiCad_LED_Chaser_202514301/
├── LED_chaser.kicad_pro          # File quản lý cấu hình dự án & Design Rules / Netclasses
├── LED_chaser.kicad_sch          # Sơ đồ nguyên lý (ERC: 0 lỗi, 0 cảnh báo)
├── LED_chaser.kicad_pcb          # Thiết kế mạch in PCB 2 lớp F.Cu & B.Cu (DRC: 0 lỗi)
├── LED_Chaser_Sch.kicad_sym      # Thư viện Symbol nguyên lý cục bộ của dự án
├── sym-lib-table                 # Bảng khai báo thư viện Symbol (${KIPRJMOD})
├── LED_Chaser_Fp.pretty/         # Thư viện Footprint linh kiện xuyên lỗ (THT) cục bộ
├── fp-lib-table                  # Bảng khai báo thư viện Footprint (${KIPRJMOD})
└── output/                       # Thư mục xuất file phục vụ sản xuất & lắp ráp
    ├── gerber/                   # Bộ 9 lớp Gerber, file .gbrjob và file khoan LED_chaser.drl
    ├── bom/                      # Thống kê vật tư linh kiện (LED_chaser_BOM.csv)
    └── pick_and_place/           # File tọa độ lắp ráp linh kiện (.csv và .pos)
```

### Chi tiết các thành phần chính
- **`LED_chaser.kicad_pro`**: Cấu hình luật thiết kế tối thiểu (`clearance >= 0.25 mm`, `track_width >= 0.25 mm`) và phân chia 2 Netclass (`Default` cho tín hiệu, `Power` `0.8 mm` cho mạng `+9V` và `GND`).
- **`LED_chaser.kicad_sch`**: Sơ đồ nguyên lý hoàn chỉnh, kiểm tra quy tắc điện (`kicad-cli sch erc`) đạt **0 Errors, 0 Warnings**.
- **`LED_chaser.kicad_pcb`**: Bố trí 20 linh kiện trên bo mạch 2 lớp (`F.Cu` & `B.Cu`) kích thước `76.2 mm × 41.0 mm`, phủ đồng `GND` cả 2 mặt với kết nối Thermal Relief, kiểm tra thiết kế (`kicad-cli pcb drc --schematic-parity`) đạt **0 DRC violations, 0 unconnected pads, 0 schematic parity issues**.
- **`LED_Chaser_Sch.kicad_sym` & `LED_Chaser_Fp.pretty/`**: Thư viện riêng đi kèm trong project, khai báo bằng đường dẫn tương đối `${KIPRJMOD}` qua `sym-lib-table` và `fp-lib-table` để đảm bảo tính độc lập khi chia sẻ/nén dự án.
- **`output/`** (chứa dữ liệu sản xuất chuẩn hóa):
  - `output/gerber/`: Đầy đủ 9 lớp Gerber (`F.Cu`, `B.Cu`, `F.Paste`, `B.Paste`, `F.Silkscreen`, `B.Silkscreen`, `F.Mask`, `B.Mask`, `Edge.Cuts`), `LED_chaser-job.gbrjob` và file lỗ khoan Excellon `LED_chaser.drl`.
  - `output/bom/`: File thống kê linh kiện `LED_chaser_BOM.csv`.
  - `output/pick_and_place/`: File tọa độ vị trí linh kiện `LED_chaser-all-pos.csv` và `LED_chaser-all-pos.pos`.
