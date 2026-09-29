# CleanFlow – CONTEXT (DENSO Factory Hacks 2026, đề H3)

Tài liệu tổng hợp để nạp vào context cho người hoặc AI tiếp tục dự án. Cập nhật lần cuối: 29/09/2026.
Đây là trạng thái đã chốt. Muốn đổi hướng thì sửa file này trước.

---

## 1. Dự án trong một đoạn

DENSO sản xuất theo kiểu high-mix low-volume: rất nhiều loại linh kiện nhỏ, mỗi loại số lượng ít. Know-how gắp–đặt (kẹp ở đâu, mở tay bao nhiêu, hạ nhanh hay chậm) chỉ nằm trong tay người thợ.

**CleanFlow** đo know-how đó từ video người thợ và lưu thành **Skill Recipe**: một file YAML đọc được và sửa được. Cobot FR5 với gripper 2 ngón chạy lại theo recipe. Đổi part thì chỉ đổi file recipe. Camera nhận diện part, kiểm tra kết quả, và mọi cycle được ghi log để tính KPI và ROI.

**Căn cứ từ DENSO:** DENSO nói khoảng một nửa số dự án tự động hóa không triển khai được vì thiếu chuyên gia, và công việc high-mix "không có quy trình chuẩn, cần kỹ năng và kinh nghiệm". Nguồn: DENSO DRIVEN BASE, bài *Symbiotic robot*: https://www.denso.com/global/en/driven-base/tech-design/symbiotic_robot/

**Mốc thời gian:**
- 12/10: nộp Ý TƯỞNG (deck theo template DENSO cùng 4 artifact minh họa).
- Nếu vào top 10: thuyết trình 16/11.
- Nếu vào top 5: chung kết 2/12.

**Đề H3 yêu cầu và cách CleanFlow đáp ứng:**

| H3 yêu cầu | CleanFlow đáp ứng bằng |
|---|---|
| Dataset know-how định lượng | Video người thạo tay và người mới, lấy số bằng `vision/extract.py`, lưu `*_reps.csv` và recipe |
| Demo tự động hóa chi phí thấp | FR5 và gripper 2 ngón (đã có sẵn), tấm đế in 1:1, webcam, ống co nhiệt |
| Báo cáo Cost/ROI | `analysis/kpi.py` (số đo được) và bảng ROI (Measured / Assumption / Target) |

## 2. Quyết định đã chốt (đừng mở lại nếu không có lý do mới)

1. **Định vị là Skill Recipe.** Bỏ hướng "mobile manipulator giữa các trạm rửa". Robot di động chỉ là hướng mở rộng, không nói là đã làm.
2. **Không để robot tự học end-to-end từ video.**
   - Know-how sẽ bị giấu trong trọng số, trái với yêu cầu "định lượng" của H3.
   - Học từ tay người sang gripper vẫn là bài toán nghiên cứu, không kịp làm ổn định trong 2 tuần.
   - Cách làm thay thế là *Learning from Demonstration → tham số đọc được*: video → số → recipe.
3. **Không dùng YOLO trước 12/10.** Ốc đứng trong lỗ đồ gá nên vị trí đã biết trước. Vision chỉ làm 3 việc:
   - `extract`: video → recipe;
   - `identify`: đo kích thước bằng OpenCV để tự chọn recipe;
   - `verify`: kiểm tra có ốc trong lỗ hay không.

   YOLO để dành cho vòng 16/11 (part nằm lộn xộn trên khay).
4. **Đo lực tối giản:**
   - Lực cắm: đặt đồ gá đặt lên cân nhà bếp và quay màn hình cân; phía robot dùng thêm torque khớp (lấy mốc rồi tính độ lệch).
   - Lực kẹp: quét trực tiếp trên robot bằng `grip_sweep.py`.
   - Không dùng FSR, Arduino hay load cell trước 12/10.
5. **Part thử nghiệm là ốc, đứng đầu hướng lên trong lỗ đồ gá:**
   - A: M5×20 lục giác, kẹp vào đầu.
   - B: M4×12 đầu tròn, kẹp thân ngay dưới đầu, hạ chậm.
   - C: M3×8, là "part chưa thấy" để thử onboard.
6. **Hero number:** số phút để đưa Part C vào chạy (quay video → extract → recipe → cycle PASS đầu tiên), do một người ngoài nhóm làm. So với thời gian lập trình variant A từ con số không.
7. **Định lượng know-how:** so sánh người thạo tay với người mới, mỗi người 10 lần cho mỗi variant. Dùng `analysis/kpi.py --compare`.
8. **Trung thực trong deck:**
   - Mọi số đều gắn nhãn Measured / Assumption / Target / Proposed, kèm n.
   - Không gọi là "AI" cho phần rule-based. Chỉ `extract.py` (MediaPipe) mới là ML.
   - Chi phí proof-of-concept của nhóm (vài trăm nghìn đồng) tách riêng khỏi chi phí DENSO triển khai thật (DENSO vẫn phải mua cobot).
   - Recipe nên không phụ thuộc hãng robot. DENSO có robot riêng (DENSO WAVE, ví dụ COBOTTA).
9. **Bằng chứng năng lực nhóm** (từ dự án robot vẽ): đã phân loại lực tiếp xúc bằng torque khớp FR5, không dùng cảm biến lực. Dữ liệu 759 mẫu, 17 file. Độ chính xác khi giữ nguyên từng file để test là **~76%**, trong khi đoán theo lớp đông nhất là 43%. Không dùng con số 79% (tính trên cách chia ngẫu nhiên, bị rò dữ liệu).

## 3. Phần cứng bắt buộc (chi phí PoC khoảng 200–450 nghìn đồng, tính thêm những thứ đã có)

| Thứ | Ghi chú |
|---|---|
| FR5 + gripper 2 ngón | Có sẵn |
| Ốc A/B/C, mỗi loại 20–30 con | |
| Tấm đế cứng và phẳng ≥ 45×32 cm + 2 kẹp chữ C | Bắt chặt vào bàn robot |
| Bản in `tools/cleanflow_board_A3.pdf` | In A3, 100%; đo 2 thước 100 mm |
| Máy khoan, mũi 3.5 / 4.0 / 4.5 / 5.0 / 5.5 / 6.0 mm | Dán cờ băng keo trên mũi để khoan đúng độ sâu |
| Ống co nhiệt (thay pad silicon) | Hoặc khối có rãnh chữ V |
| Webcam USB 1080p, hoặc điện thoại dùng DroidCam/Iriun | Máy tính phải đọc được hình trực tiếp |
| Điện thoại thứ hai + giá kẹp | Quay góc ngang khi người làm mẫu |
| **Thước kẹp (caliper)** | Mọi kích thước trong recipe và config |
| Cân nhà bếp (độ chia 1 g) | Cân ốc, đo lực cắm |
| Đèn LED, giấy nền mờ màu tương phản, băng keo màu cho móng tay | |

## 4. Khung tọa độ và tấm đế

- **Khung board:** gốc là tâm marker ArUco **ID3** (góc dưới-trái, phía robot). **+X** hướng về ID2, **+Y** hướng về ID0, Z hướng lên, Z = 0 là mặt tấm đế. Đơn vị mm.
- **Marker:** DICT_4X4_50, cạnh 40 mm. Tâm các marker: ID0 (0,240), ID1 (360,240), ID2 (360,0), ID3 (0,0). **Không** để file `markers_input_40mm.pdf` cũ trong khung hình, vì nó cũng dùng ID 0–3 và detector sẽ lẫn giữa hai bộ.
- **Đồ gá:** mỗi đồ gá 4 cột, lỗ cách nhau 30 mm. Đồ gá lấy bắt đầu ở x0 = 40, đồ gá đặt ở x0 = 230. Hàng A ở y = 150, B ở y = 120, C ở y = 90. Ô nhận diện có tâm (175,120), cạnh 40 mm.

  | Hàng | Lỗ lấy (rộng) | Lỗ đặt (chặt) | Sâu |
  |---|---|---|---|
  | A | Ø6.0 | Ø5.5 | 10 mm |
  | B | Ø5.0 | Ø4.5 | 5 mm |
  | C | Ø4.0 | Ø3.5 | 3 mm |
- **Nguồn dữ liệu duy nhất là `config.yaml`.** Sửa hình học trong đó rồi chạy `python tools/make_board.py` để in lại.
- **User frame trên FR5:** dạy bằng 3 điểm: tâm ID3 (gốc), tâm ID2 (+X), tâm ID0 (mặt XY). Điền số hiệu vào `robot.user`.
- **Tool frame:** TCP đặt ở **tâm giữa hai đầu ngón**. Điền vào `robot.tool`.

**Quy tắc hình học** (`common/geometry.py` kiểm tra tự động):
- Ốc đứng trong lỗ sâu D: đỉnh đầu ốc ở z = chiều cao đầu + max(0, dài thân − D).
- Recipe mô tả điểm kẹp bằng `grip.depth_from_head_top_mm`, tức khoảng cách từ đỉnh đầu ốc xuống tâm đầu ngón. Camera ngang đo đúng đại lượng này.
- Mép dưới đầu ngón phải cao hơn mặt đế ≥ `min_finger_clearance_mm`. Nếu không, recipe bị từ chối. Vì thế **lỗ phải nông hơn thân ốc**.
- Độ cao di chuyển tự tính sao cho đầu ốc đang cầm vượt qua mọi ốc đang đứng ít nhất 5 mm.

## 5. Cấu trúc code

```
cleanflow/
  CONTEXT.md            ← file này
  config.yaml           ← mọi tham số: robot, an toàn, gripper, torque, tấm đế, đồ gá, camera
  recipes/A.yaml B.yaml C.yaml   ← SEED (chưa đo). Thay bằng bản extract.py sinh ra
  common/config.py      ← đọc config, sinh tọa độ lỗ, kiểm tra recipe (kiểu dữ liệu, trần tốc độ/lực, hình học)
  common/geometry.py    ← tính các độ cao Z: kẹp / chạm miệng lỗ / cắm xong / di chuyển
  robot/fr5.py          ← wrapper SDK Fairino + SafetyGuard + MockRobot (chạy thử không cần robot)
  robot/cell.py         ← state machine gắp–cắm, retry, xử lý kẹt, ghi log CSV
  robot/run_recipe.py   ← CLI chạy cycle (mock / sim / real, --step, --identify, --verify, --return-trip)
  robot/torque_log.py   ← TorqueWatch (lấy mốc + độ lệch), xem torque trực tiếp. Thay do_luc.py và joint_torque.py
  robot/grip_sweep.py   ← hiệu chuẩn lúc đóng rỗng + quét lực kẹp → gợi ý force_pct
  robot/teach_tpd.py    ← kéo tay FR5 và ghi quỹ đạo TPD → phát lại (nguồn know-how thứ 2)
  vision/board.py       ← ArUco → homography ảnh↔mm, nắn ảnh nhìn từ trên, hiệu chỉnh thị sai
  vision/camera.py      ← đọc webcam (bỏ các khung hình cũ trong bộ đệm)
  vision/extract.py     ← video → MediaPipe Tasks HandLandmarker → tracks, reps, recipe nháp
  vision/features.py    ← tách sự kiện kẹp/thả, tính số liệu từ tracks (thuần numpy, có test)
  vision/identify.py    ← đo dài/rộng part trong ô nhận diện → A/B/C → recipe
  vision/verify.py      ← có ốc trong lỗ đích không (so với ảnh tham chiếu)
  analysis/kpi.py       ← cycles.csv → bảng KPI (markdown/CSV), changeover, gợi ý ngưỡng kẹt; --compare demo
  tools/make_board.py   ← sinh PDF tấm đế A3 từ config.yaml (+ bản PDF đã sinh)
  legacy/eval_torque_classifier.py ← đánh giá lại mô hình torque của dự án vẽ (bằng chứng năng lực)
  fairino/Robot.py      ← SDK Fairino (Apache-2.0), lấy từ bản của nhóm
  tests/test_all.py     ← 12 test (mock robot, ảnh tấm đế, video tổng hợp)
  models/               ← đặt hand_landmarker.task ở đây (xem README.txt)
```

**Lệnh chạy chính:**
```
pip install -r requirements.txt
python -m pytest -q                                              # 12 test, không cần phần cứng
python -m robot.run_recipe --variant B --cycles 4 --mode mock    # chạy thử logic
python -m robot.grip_sweep --empty                               # → gripper.empty_close_pct
python -m robot.grip_sweep --variant B --forces 10 20 30 40 --reps 5
python -m vision.extract --video demos/B_expert_top.mp4 --variant B --grip-depth-mm 5.0 --out recipes/B.yaml
python -m vision.identify                                        # thử nhận diện
python -m robot.run_recipe --identify --cycles 4 --verify --return-trip
python -m analysis.kpi
python -m analysis.kpi --compare demos/B_expert_top_reps.csv demos/B_novice_top_reps.csv
```

**Log sinh ra:**
- `logs/cycles.csv`, mỗi cycle một dòng: cycle_id, timestamp, variant_id, recipe_version, direction, pick_slot, place_slot, pick_verified, placement_verified, cycle_time_s, retry_count, final_status, fail_state, grip_pos_pct, grip_current_pct, insert_peak_dtorque_nm.
- `logs/torque/cycle_XXXX.csv`
- `logs/events.csv` (thời điểm nạp recipe, dùng để tính changeover)

## 6. An toàn: đã có gì và quy trình chạy robot thật lần đầu

**Có sẵn trong code:**
- `SafetyGuard` kiểm tra **mọi** điểm đến: nằm trong hộp tấm đế ± margin, z_min ≤ z ≤ z_max, không có NaN.
- Trần tốc độ (`max_vel_pct`, `max_insert_vel_pct`) và trần lực kẹp. Recipe vượt trần sẽ bị từ chối ngay khi nạp.
- **Giám sát torque:**
  - Đoạn hạ cuối chạy non-blocking dưới ngưỡng `unexpected_contact_nm`; vượt ngưỡng thì StopMotion.
  - Đoạn cắm dùng ngưỡng `insert.jam_dtorque_nm` của recipe. Khi kẹt, robot **không thả ốc**: rút ra chậm, đem về lỗ lấy, rồi mới thả.
- Gắp trượt thì thử lại `max_grasp_retries` lần rồi bỏ qua lỗ đó. Phát hiện gắp trượt dựa vào vị trí hàm so với `empty_close_pct`.
- Chỉ một luồng gửi lệnh (xmlrpc không an toàn khi nhiều luồng gửi). Torque được đọc từ gói trạng thái.
- Timeout cho mỗi chuyển động thì StopMotion. Ctrl+C thì StopMotion. Mọi lỗi SDK đều dừng và ghi log.
- `mode: real` bắt gõ **RUN**. `--step` dừng chờ Enter trước từng chuyển động. `mode: mock` chạy logic không cần robot.
- Khi kết nối: từ chối chạy nếu controller đang báo lỗi. Có thể đặt mức va chạm và chiến lược xử lý va chạm (mặc định null, tức giữ cài đặt trên WebApp).

**Quy trình lần đầu:**
1. `pytest` phải xanh hết.
2. Chạy mock.
3. Chạy trên simulator.
4. Robot thật: `global_speed_pct: 20`, thêm `--step`, `--cycles 1`, một người đứng cạnh E-stop, không có ai trong tầm với của robot.
5. Kiểm tra TCP và user frame bằng cách cho robot tới (x, y, 30) của lỗ PA1: đầu ngón phải nằm đúng trên tâm lỗ.
6. Chạy `grip_sweep --empty` rồi mới bật phát hiện gắp trượt.
7. Tăng tốc độ dần.

## 7. Sự thật về SDK Fairino (đã kiểm tra trong `fairino/Robot.py` của nhóm)

- **`MoveGripper(..., block, ...)`: `0` = BLOCKING, `1` = NON-blocking.** Ngược với trực giác. (Trong trao đổi trước có chỗ nói ngược. Code dùng đúng.)
- `MoveL(... blendR=-1)` là blocking. `blendR ≥ 0` là non-blocking, cần tự chờ `GetRobotMotionDone`.
- `vel` của MoveL tính theo %, còn bị nhân thêm với `SetSpeed` toàn cục.
- `GetJointTorques`, `GetActualTCPPose`, `GetRobotMotionDone`, `GetGripperCurPosition`, `GetGripperCurCurrent`, `GetRobotErrorCode` đọc từ **gói trạng thái real-time**, không đi qua XML-RPC, nên đọc nhanh được.
- `GetGripperCurPosition()` trả về `(err, fault, pos)`. Hàm này trả về 3 giá trị, không phải 2.
- TPD: `SetTPDParam` → `SetTPDStart(name, period_ms ∈ {2,4,8})` → `SetWebTPDStop` → `LoadTPD` → `GetTPDStartPose` → `MoveCart` → `MoveTPD(name, blend, ovl)`.
- `SetAnticollision(mode, level[6], config)`: mode 0 dùng thang 1–10. `SetCollisionStrategy`: 0 = báo lỗi và tạm dừng, 2 = báo lỗi và dừng.
- IP mặc định của controller là `192.168.58.2`. Máy của nhóm từng dùng `10.12.3.18`. Cả hai được gom vào `config.yaml`.
- **mediapipe 0.10.31 KHÔNG còn `mp.solutions`.** Phải dùng `mediapipe.tasks.python.vision.HandLandmarker` và file model `.task`.

## 8. Kiểm kê thư mục SDK / robot vẽ: Giữ, Nâng cấp hay Bỏ

| File gốc | Quyết định | Đi đâu / lý do |
|---|---|---|
| `fairino/Robot.py` | **Giữ** | `cleanflow/fairino/Robot.py` |
| `do_luc.py`, `joint_torque.py` | **Nâng cấp** | `robot/torque_log.py`: lấy mốc tại đúng tư thế, đọc 100 Hz, ghi CSV |
| `monitor_torque()` trong `251225_draw_bot.py` | **Nâng cấp** (sửa lỗi) | Có `break` trong vòng lặp nên chỉ đọc 1 lần; `sleep(0)` làm vòng lặp chạy hết CPU. Nay là `TorqueWatch` được poll trong `wait_motion` |
| Logger Excel trong `251225_draw_bot.py` | **Nâng cấp** | `CycleLogger`, ghi CSV theo schema KPI |
| `example/TestTPDCommand.py` | **Nâng cấp** | `robot/teach_tpd.py`, có bước xác nhận và giới hạn tốc độ |
| `example/TestPeripheralsCommand.py` (gripper) | **Nâng cấp** | `FairinoRobot.gripper()` và `grip_sweep.py` |
| `test_moveL.py` | **Tham khảo, bỏ** | Tham số MoveL đã nằm trong `fr5.py` |
| `train_from_real_data.py`, `model_luc.py`, `model/*.pkl`, `data/*.xlsx` | **Chỉ giữ làm bằng chứng năng lực** | `legacy/eval_torque_classifier.py` (đánh giá đúng cách). Không dùng mô hình này cho ốc |
| `.venv/` (1,2 GB), `fairino/build/`, `Robot.c`, `libfairino/*.pyd`, `__pycache__` | **Bỏ** | Đường dẫn gắn cứng với máy khác và file sinh ra khi build. Cài lại bằng `requirements.txt` |
| `gcode/`, `gcode_to_robot(2).py`, `(2.2)`, `251225_draw_bot.py`, `robot_draw_animation_fr5.py`, `test_di_chuyen.py` | **Bỏ khỏi dự án DENSO** | Thuộc dự án robot vẽ, để ở repo riêng |
| Phần còn lại của `example/` (~97 file) | **Bỏ** | Tra lại trong SDK gốc khi cần |
| `torch`, `sounddevice`, `svgpathtools`, `svgwrite`, `scikit-image` | **Bỏ khỏi requirements** | Không script nào dùng |
| `markers_input_40mm.pdf` | **Bỏ** | Thay bằng tấm đế A3; trùng ID 0–3 |

## 9. Trạng thái và việc còn phải làm

- **Đã kiểm thử:** 12/12 test trên MockRobot, trên ảnh render từ PDF tấm đế (homography, identify, verify) và trên video tay tổng hợp (features → recipe).
- **CHƯA chạy trên simulator hay FR5 thật.** Đợt chạy thật đầu tiên phải theo đúng mục 6.
- **CHƯA chạy `extract.py` trên video thật** (cần tải file `.task`). Mới kiểm thử phần tách sự kiện trên dữ liệu tổng hợp.
- **Phải điền trong `config.yaml`** (chỗ ghi CHANGE_ME):
  - `ip_sim`, `tool`, `user`, `tool_orientation_deg`;
  - `gripper.pct_closed/pct_open/stroke_mm/finger_half_height_mm`, `empty_close_pct`;
  - `depth_mm` thật của từng hàng lỗ;
  - `camera.height_mm`.
- **Recipe A/B/C hiện là SEED.** Phải thay bằng bản sinh từ video người thạo tay. Riêng C phải được tạo mới trong thử nghiệm Part C.
- **Ngưỡng kẹt `jam_dtorque_nm`:** hiệu chỉnh theo `analysis/kpi.py` (gợi ý = 1,5 × đỉnh lớn nhất của các cycle PASS).
- **Deck:** 8 slide của template DENSO + 1 slide bằng chứng. Checklist 6 câu hỏi có số liệu. Câu pitch chốt theo mẫu: *"…đưa một linh kiện mới vào tự động hóa trong X phút thay vì Y giờ…"*.
