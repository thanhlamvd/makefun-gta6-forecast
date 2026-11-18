# Dự báo lượng người xem GTA VI trên Twitch theo từng giờ (ngày ra mắt)

[![make.fun](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01)
[![Rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/thanhlamvd/makefun-gta6-forecast/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01)
[![Reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/thanhlamvd/makefun-gta6-forecast/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01)

Bài dự thi cho bounty [Forecast GTA VI's launch-day Twitch viewers hour by hour](https://make.fun/bounty/gta6-twitch-ccu-forecast-01) trên make.fun.

Grand Theft Auto VI ra mắt ngày 19/11/2026 (chỉ trên PS5 và Xbox Series X|S, chơi đơn nên không có số người chơi trực tuyến công khai). Bài này dự báo, theo từng giờ (giờ 2–24, giờ UTC) của ngày ra mắt, có bao nhiêu người đang xem mục "Grand Theft Auto VI" trên Twitch — dưới dạng 5 phân vị (5%, 25%, 50%, 75%, 95%) cho mỗi giờ.

## Cách chạy

```sh
mkdir -p out
MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

Chương trình đọc mọi file `<id>.in` trong thư mục input và ghi `<id>.out` tương ứng ra thư mục output. Chỉ dùng thư viện chuẩn Python 3.13, không mạng, không ngẫu nhiên, không đọc đồng hồ — cùng một input luôn cho ra đúng một output (đã kiểm tra byte-identical trên 2 lần chạy), thời gian chạy ~0,2 giây cho 4 case thực hành (giới hạn là 8 giây/case).

## Phương pháp

Tất cả tính toán đều làm trong không gian log (vì điểm số là pinball loss trên thang log).

1. **Neo vào giờ đầu tiên.** Trung vị dự báo của mọi giờ đều neo vào số người xem của giờ 1 (`known`, giờ 00:00–01:00 UTC ngày 19/11). Nếu giờ 1 không dùng được thì lùi về điểm giờ cuối cùng trước checkpoint còn dương, cuối cùng là 1 người xem.

2. **Đường cong trong ngày học từ chính các comparable trong case.** Mỗi comparable là một game lớn ra mắt trước đây với số liệu Twitch theo giờ (14 ngày trước tới 3 ngày sau ngày ra mắt). Với mỗi comparable, lấy log(v_h / v_1) — tỉ lệ giờ h so với giờ 1 của chính ngày ra mắt của nó — rồi lấy **trung vị** qua các comparable làm đường cong m_h: giờ đầu trũng xuống (lúc đó phần lớn thế giới còn đang ngủ), sau đó leo dần lên đỉnh buổi tối (giờ 19–21 UTC, khoảng 3,3 lần giờ 1) rồi hạ dần. Dùng trung vị thay vì trung bình để một lần ra mắt dị thường không kéo lệch đường cong.

   Hai bộ lọc giữ cho pool so sánh "giống" GTA VI — game console mở khóa lúc nửa đêm theo giờ địa phương, lăn theo từng quốc gia:
   - loại lịch sử pre-release của chính mục GTA VI (không phải một lần ra mắt);
   - loại các game early-access, free-to-play hoặc tier yếu (động lực ngày đầu của chúng khác hẳn một lần mở khóa nửa đêm).

3. **Độ rộng khoảng tin cậy theo mức độ bất đồng giữa các lần ra mắt.** Độ lệch chuẩn s_h mỗi giờ bằng 0,55 × độ lệch chuẩn quần thể của các log-ratio ở giờ đó, kẹp trong [0,25; 0,90]. Giờ nào các lần ra mắt bất đồng nhiều thì dải dự báo rộng; càng về cuối ngày các lần ra mắt càng phân kỳ nên dải cũng rộng dần. Giờ nào quá ít comparable thì dùng độ rộng dự phòng 0,30 + 0,08·√(h−1).

4. **Phân vị.** q_τ(h) = p1 · exp(m_h + z_τ · s_h), với z_τ là phân vị chuẩn (xấp xỉ Acklam), làm tròn nửa lên tới 2 chữ số thập phân, kẹp trong (0; 10⁹] và ép không giảm trong mỗi hàng. Trung vị (τ = 50%) chính là p1 · exp(m_h) — tức đường cong hình dạng.

## Kiểm định offline (leave-one-out)

Vì đáp án của các case thực hành do máy chủ giữ, không tự chấm practice được. Thay vào đó, kiểm định bằng leave-one-out trên 37 comparable đi kèm case thực hành (dự báo từng lần ra mắt từ 36 lần còn lại, chấm pinball loss trên thang log so với số thực tế):

| Mô hình | Loss trung bình | 90% coverage |
|---|---|---|
| House baseline (trung vị phẳng, độ rộng 0,15·√(h−1)) | ~0,930 | ~0,62 |
| **Mô hình này** | **~0,800** | **~0,81** |

Trên 8 lần ra mắt "giống GTA VI nhất" (mở khóa nửa đêm theo giờ địa phương, không early-access): baseline ~0,418, mô hình này ~0,375, coverage ~0,94. Cải thiện có thật nhưng khiêm tốn — đúng kỳ vọng cho một bài toán mà độ bất định giữa các lần ra mắt là rất lớn.

## Giới hạn

- Dự báo chỉ tốt bằng đường cong trung vị của các lần ra mắt trước; nếu GTA VI phá vỡ khuôn mẫu (ví dụ đỉnh xem vượt xa mọi lần ra mắt từng có), trung vị sẽ trệch và chỉ có dải phân vị rộng cứu điểm.
- Tín hiệu pre-release (lịch sử mục GTA VI trước ngày ra mắt, top stream) không mang thông tin về mức độ của ngày ra mắt ngoài giờ 1 nên chương trình cố tình không dùng chúng để dịch chuyển mức dự báo — chỉ dùng làm dự phòng khi thiếu giờ 1.
- Không có practice score: máy chủ mới chấm được case thực hành, nên con số trên là kiểm định leave-one-out trên dữ liệu comparable, không phải điểm practice.

## Giấy phép

MIT — xem [LICENSE](LICENSE).
