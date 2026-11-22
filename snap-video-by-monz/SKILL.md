---
name: snap_video_by_monz
description: Dựng một Không gian (Space) tải video tên "Snap video by Monz" trong Muse của chính người dùng — dán link TikTok, Douyin, YouTube, Facebook, Bilibili, Instagram hoặc X là bóc được video gốc sạch watermark, chọn chất lượng tới 4K, tải về dạng MP4 chuẩn H.264 + AAC để iPhone lưu thẳng vào thư viện Ảnh. Backend yt-dlp + ffmpeg chạy trên máy chủ của Space, tối đa 80MB mỗi video, chỉ tải video công khai. Dùng khi người dùng yêu cầu cài / dựng lại tool Snap video by Monz từ gói này.
---

# Snap video by Monz — Tool tải video từ link

## Mục đích

Người dùng dán link video từ TikTok, Douyin, YouTube, Facebook, Bilibili, Instagram
hoặc X → tool bóc ra video gốc **sạch watermark**, cho chọn chất lượng, tải về máy
dạng MP4 xem trước được ngay, và lưu lại vào lịch sử "Đã snap" để tải lại sau.

Tool này sống dưới dạng **một Không gian (Space) dạng fullstack** trong Muse của
người dùng (có backend và cơ sở dữ liệu riêng của Space). Mỗi người cài gói này
sẽ có một bản riêng, lịch sử trống, chạy bằng tài khoản Muse của chính họ.

## Việc Muse nhận gói này phải làm

Khi người dùng gửi gói ZIP này và yêu cầu cài/dựng Snap video by Monz:

1. Tạo một web artifact dạng **fullstack** mới:
   - Tên hiển thị, chính xác từng chữ: `Snap video by Monz`
   - Slug gợi ý: `snap-video-by-monz`
   - Khai báo khả năng đọc web công khai (public web read), không cần connector nào.
2. Dựng đúng giao diện, chức năng và các luật cứng ở dưới.
3. Kiểm chứng thật bằng một link video công khai (xem mục Kiểm chứng) rồi mới giao.

## Giao diện (đúng nguyên văn)

- Tiêu đề lớn: **Snap video** (chữ đen đậm) + **by Monz** (chữ đỏ) trên cùng một dòng.
- Dòng giới thiệu dưới tiêu đề, nguyên văn:
  `Dán link là bóc ra video gốc — sạch watermark, chọn được chất lượng cao nhất.`
- Hàng chip nền tảng: `TikTok` `Douyin` `YouTube` `Facebook` `Bilibili` `Instagram` `X`
- Ô nhập link, placeholder nguyên văn:
  `Dán link TikTok, Douyin, YouTube, Facebook, Bilibili...`
- Hai nút cạnh nhau: `Dán link` (dán từ clipboard) và nút đỏ `SNAP →`
- Sau khi resolve: thẻ kết quả gồm ảnh bìa (thumbnail), tiêu đề, kênh/người đăng,
  thời lượng, và danh sách mức chất lượng dạng radio để chọn trước khi tải.
- Mục lịch sử, tiêu đề nguyên văn: `Đã snap`
  - Trạng thái trống, nguyên văn: `Chưa snap video nào. Dán link đầu tiên ở trên đi.`
  - Mỗi mục gồm: thumbnail, tiêu đề, dung lượng, các nút `Lưu MP4`, `Tải lại`, `Xoá`.
  - **KHÔNG thêm bất kỳ dòng hướng dẫn/ghi chú nào dưới tiêu đề "Đã snap".**
- Dòng chân trang, nguyên văn:
  `Chạy bằng yt-dlp + ffmpeg · tối đa 80MB mỗi video · chỉ tải video công khai.`
- Ngôn ngữ giao diện: tiếng Việt. Tối ưu cho iPhone (mobile-first), không tràn ngang.

## Backend

- Dùng **yt-dlp** để resolve link: lấy tiêu đề, người đăng, thumbnail, thời lượng
  và danh sách format (từ thấp tới cao, tối đa 4K nếu nguồn có).
- Dùng **yt-dlp + ffmpeg** để tải format người dùng chọn: nếu nguồn phát hình và
  tiếng tách riêng thì ffmpeg ghép lại thành một file MP4 duy nhất.
- Giới hạn cứng: **tối đa 80MB mỗi video** — format nào vượt thì không cho chọn
  hoặc báo rõ trước khi tải.
- **Chỉ tải video công khai.** Video riêng tư / cần đăng nhập / bị khoá vùng mà
  yt-dlp không lấy được thì báo lỗi rõ ràng cho người dùng, không tìm cách lách.
- File video lưu vào kho blob của Space; metadata (link gốc, tiêu đề, thumbnail,
  format đã chọn, dung lượng, thời gian) lưu vào cơ sở dữ liệu của Space để hiện
  trong mục "Đã snap".

## Luật cứng về định dạng — chuẩn iPhone (quan trọng nhất)

Đây là bài học đã trả giá bằng lỗi thật, không được bỏ qua:

1. **Ưu tiên format H.264 ngay từ lúc chọn chất lượng:** trong danh sách format,
   ưu tiên codec video `avc1` (H.264) + âm thanh AAC.
2. **Sau khi tải, luôn kiểm tra codec bằng ffprobe.** Nếu video không phải H.264
   hoặc tiếng không phải AAC (rất hay gặp: YouTube phát **AV1** hoặc VP9 đóng trong
   vỏ MP4 ở các mức 1080p+), thì **chuyển mã (transcode) sang H.264 + AAC**
   bằng ffmpeg (`libx264`, `yuv420p`, `aac`, `+faststart`) trước khi lưu.
   - Lý do: file AV1/VP9 trong vỏ .mp4 nhìn vẫn là "MP4" nhưng **iPhone không
     giải mã được** — bấm "Lưu video" trong hộp chia sẻ sẽ im lặng không lưu gì,
     ô xem trước trắng xoá, không hề báo lỗi.
   - File H.264 sau chuyển mã sẽ nặng hơn bản AV1 gốc — đó là bình thường.
3. Nút **Lưu MP4** tuyệt đối KHÔNG được là một thẻ link trỏ thẳng tới URL blob:
   URL blob có tên là chuỗi mã hoá, iOS nhận file thành loại "dữ liệu" chung chung,
   không mở được, không lưu được. Cách làm đúng:
   - `fetch` blob về → đóng gói thành `File` mang tên `<slug-từ-tiêu-đề>.mp4`,
     `type: "video/mp4"`.
   - Nếu trình duyệt hỗ trợ chia sẻ file (`navigator.canShare({ files })`):
     mở `navigator.share({ files })` — trên iOS, người dùng chọn **"Lưu video"**
     là video vào thẳng thư viện Ảnh.
   - Không hỗ trợ thì fallback: tải qua object URL như file download bình thường.
   - Trạng thái lỗi / người dùng huỷ chia sẻ phải được phản ánh trên nút bấm.
4. Nút **Tải lại** ở mỗi mục lịch sử: resolve + tải lại từ link gốc, áp dụng lại
   toàn bộ luật định dạng ở trên, và **thay thế** file đã lưu của mục đó
   (để cứu các file cũ bị sai codec).

## Giới hạn nền tảng phải biết

- Mỗi lượt xử lý (action) của Space có ngân sách khoảng **30 giây**: resolve thì
  nhẹ, nhưng tải + ghép + chuyển mã phải xong trong ngân sách đó. Hệ quả:
  clip ngắn kiểu TikTok/Douyin là trường hợp dùng chính và chạy tốt; video dài
  nhiều phút ở chất lượng cao có thể quá giờ và không lưu được gì.
- Khi người dùng gặp lỗi quá giờ với video dài, hướng xử lý là khuyên chọn mức
  chất lượng thấp hơn hoặc video ngắn hơn — nói thẳng đây là giới hạn của lượt
  xử lý trong Space, không phải giới hạn của TikTok/YouTube.

## Kiểm chứng trước khi giao (bắt buộc)

- [ ] Resolve một link YouTube công khai thật: ra đúng tiêu đề, thumbnail,
      thời lượng và danh sách chất lượng.
- [ ] Tải thử một clip ngắn: luồng tải + ghép + lưu + hiện trong "Đã snap" chạy hết.
- [ ] ffprobe file kết quả: video = `h264`, audio = `aac`.
- [ ] Bấm thử nút "Lưu MP4": file tải/chia sẻ mang tên `.mp4` đàng hoàng theo
      tiêu đề video (không phải chuỗi mã hoá).
- [ ] Giao diện đúng nguyên văn các dòng chữ ở mục Giao diện, không thừa dòng
      hướng dẫn dưới "Đã snap", không tràn ngang trên mobile.
- [ ] Lịch sử bắt đầu trống — không có dữ liệu mẫu hay dữ liệu của người khác.

## Riêng tư

Gói này chỉ chứa bản đặc tả dựng tool. Không chứa dữ liệu cá nhân, lịch sử tải,
link đã snap, tài khoản, mật khẩu hay token của bất kỳ ai. Bản dựng từ gói này
luôn bắt đầu với lịch sử trống trong Muse của người cài.
