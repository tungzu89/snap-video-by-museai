---
name: snap_video_by_monz
description: Dựng một Không gian (Space) tải video tên "Snap video by Monz" trong Muse của chính người dùng — dán link TikTok, Douyin, YouTube, Facebook, Bilibili, Instagram hoặc X là bóc được video gốc sạch watermark, chọn chất lượng tới 4K, tải về dạng MP4 chuẩn H.264 + AAC để iPhone lưu thẳng vào thư viện Ảnh. Backend yt-dlp + ffmpeg chạy trên máy chủ của Space, tối đa 80MB mỗi video, chỉ tải video công khai. Dùng khi người dùng yêu cầu cài / dựng lại tool Snap video by Monz từ gói này. / Build a video-downloader Space named "Snap video by Monz" in the user's own Muse — paste a TikTok, Douyin, YouTube, Facebook, Bilibili, Instagram, or X link to extract the original watermark-free video, with quality options up to 4K, delivered as an iPhone-ready MP4 (H.264 + AAC) that saves straight to the Photos library. Server-side yt-dlp + ffmpeg backend, 80MB max per video, public videos only. Use when the user asks to install or rebuild the Snap video by Monz tool from this package. (Full English specification at the end of this file.)
---

# Snap video by Monz — Tool tải video từ link / Video Link Downloader

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

---

# ENGLISH SPECIFICATION

## Purpose

The user pastes a video link from TikTok, Douyin, YouTube, Facebook, Bilibili,
Instagram, or X → the tool extracts the original **watermark-free** video, lets
them pick a quality, downloads it as a previewable MP4, and keeps it in a
"Đã snap" (Snapped) history for later re-download.

The tool lives as a **fullstack Space** inside the user's own Muse (with its
own server backend and database). Everyone who installs this package gets their
own copy, with an empty history, running under their own Muse account.

## What the receiving Muse must do

When the user shares this package and asks to install / build Snap video by Monz:

1. Create a new **fullstack** web artifact:
   - Display name, exactly: `Snap video by Monz`
   - Suggested slug: `snap-video-by-monz`
   - Declare public web read; no connectors are needed.
2. Build the UI, features, and hard rules below — exactly.
3. Verify against a real public link (see Verification) before handing over.

## UI (exact copy)

The UI language is Vietnamese; keep these strings verbatim:

- Big title: **Snap video** (bold black) + **by Monz** (red), on one line.
- Intro line: `Dán link là bóc ra video gốc — sạch watermark, chọn được chất lượng cao nhất.`
  ("Paste a link to extract the original video — watermark-free, with the highest quality options.")
- Platform chips: `TikTok` `Douyin` `YouTube` `Facebook` `Bilibili` `Instagram` `X`
- Link input placeholder: `Dán link TikTok, Douyin, YouTube, Facebook, Bilibili...`
- Buttons: `Dán link` ("Paste link", from clipboard) and a red `SNAP →` button.
- After resolving: a result card with thumbnail, title, uploader, duration,
  and a radio list of quality options to pick before downloading.
- History section, heading: `Đã snap`
  - Empty state: `Chưa snap video nào. Dán link đầu tiên ở trên đi.`
  - Each item: thumbnail, title, file size, and `Lưu MP4` (Save MP4),
    `Tải lại` (Re-download), `Xoá` (Delete) buttons.
  - **Do NOT add any guidance/note line under the "Đã snap" heading.**
- Footer: `Chạy bằng yt-dlp + ffmpeg · tối đa 80MB mỗi video · chỉ tải video công khai.`
  ("Powered by yt-dlp + ffmpeg · 80MB max per video · public videos only.")
- Mobile-first (iPhone), no horizontal overflow.

## Backend

- Use **yt-dlp** to resolve links: title, uploader, thumbnail, duration, and
  the format list (lowest to highest, up to 4K when the source offers it).
- Use **yt-dlp + ffmpeg** to download the chosen format; when the source serves
  video and audio separately, ffmpeg merges them into a single MP4.
- Hard cap: **80MB per video** — formats above the cap are not selectable, or
  are flagged clearly before downloading.
- **Public videos only.** Private, login-gated, or region-locked videos that
  yt-dlp cannot fetch must produce a clear error — never attempt to bypass.
- Store video files in the Space's blob storage; store metadata (source URL,
  title, thumbnail, chosen format, size, timestamp) in the Space's database
  for the "Đã snap" history.

## Hard format rules — iPhone-ready output (most important)

Lessons paid for with real bugs; do not skip them:

1. **Prefer H.264 when picking formats:** prefer video codec `avc1` (H.264)
   with AAC audio in the quality list.
2. **Always probe the downloaded file with ffprobe.** If the video is not
   H.264 or the audio is not AAC (very common: YouTube serves **AV1** or VP9
   inside an MP4 container at 1080p+), **transcode to H.264 + AAC** with ffmpeg
   (`libx264`, `yuv420p`, `aac`, `+faststart`) before storing.
   - Why: an AV1/VP9 file in an .mp4 wrapper still looks like "MP4", but
     **iPhones cannot decode it** — tapping "Save Video" in the share sheet
     silently saves nothing, with a blank preview and no error.
   - A transcoded H.264 file will be larger than the AV1 original. That is normal.
3. The **Save MP4** button must NEVER be a plain anchor to the blob URL: blob
   URLs carry an encoded name, so iOS receives the file as generic "data" that
   cannot be opened or saved. Correct approach:
   - `fetch` the blob → wrap it in a `File` named `<title-slug>.mp4`,
     `type: "video/mp4"`.
   - If file sharing is supported (`navigator.canShare({ files })`), open
     `navigator.share({ files })` — on iOS the user picks **"Save Video"**
     and the video lands in the Photos library.
   - Otherwise fall back to a normal object-URL download.
   - Errors and user cancellations must be reflected in the button's state.
4. The **Re-download** button on each history item re-resolves and re-downloads
   from the source URL, re-applies all format rules above, and **replaces**
   that item's stored file (this rescues older files with the wrong codec).

## Platform limits to know

- Each Space action has a budget of roughly **30 seconds**: resolving is light,
  but download + merge + transcode must finish inside that budget. Short
  TikTok/Douyin-style clips are the primary use case and work well; long,
  high-quality videos may time out and save nothing.
- When a long video times out, advise a lower quality or a shorter video —
  and state plainly that this is the Space's per-action limit, not a
  TikTok/YouTube limit.

## Verification (mandatory before hand-over)

- [ ] Resolve a real public YouTube link: correct title, thumbnail, duration,
      and quality list.
- [ ] Download a short clip end to end: download + merge + store + appears in "Đã snap".
- [ ] ffprobe the result: video = `h264`, audio = `aac`.
- [ ] Test the "Lưu MP4" button: the shared/downloaded file has a proper
      title-based `.mp4` name (not an encoded string).
- [ ] UI copy matches the spec verbatim, no extra guidance line under
      "Đã snap", no horizontal overflow on mobile.
- [ ] History starts empty — no sample data, nobody else's data.

## Privacy

This package contains only the build specification. It contains no personal
data, download history, snapped links, accounts, passwords, or tokens.
Any build from this package starts with an empty history inside the
installer's own Muse.
