# Ghi chú: làm lyric video theo cách của pdoom-video

Ngày ghi chú: 09/10/2026. Bản mã nguồn tham chiếu: `fa164b74fd0b26a99375715b619c55b4a0fb3d05`.

Đây là ghi chú từ cuộc trao đổi và các nguồn đã kiểm tra, để chuẩn bị một video mới.
Chưa có nhạc, lời hoặc brief riêng cho video mới; tài liệu này chưa phải một bản dựng.

## 1. Video được tạo từ những gì?

Video ghép **bài nhạc + lời có timing + kịch bản hình ảnh + code animation**.
Chữ chạy theo giọng hát; chuyển động và chuyển cảnh được thiết kế theo ý nghĩa lời và các điểm nhấn âm nhạc.

| Thành phần | Nguồn và vai trò |
| --- | --- |
| Lời bài hát | MusicPerson viết phần mở đầu và điệp khúc; osmarks phát triển thêm lời với sự hỗ trợ của Claude. Trang osmarks cung cấp lời chuẩn và diễn giải. |
| Bản nhạc gốc | Được tạo bằng Udio và phát hành tháng 11/2024, theo credits của dự án. |
| Nhạc trong video | Bản “Claude-Pop” tạo bằng Suno, do deckard (`@slimer48484`) đăng tháng 9/2026. |
| Timing từng từ | Phân tích bản audio được chọn, đối chiếu với lời đã biết, rồi lưu trong `data/lyrics.json`. |
| Nhịp và đặc trưng âm thanh | Script phân tích audio tạo dữ liệu trong `data/audio.json` để scene sử dụng. |
| Ý tưởng hình ảnh | Cuộc trao đổi giữa tác giả và Claude, được cụ thể hóa thành treatment, bảng màu, font, motif và các cảnh. |
| Animation | Code TypeScript, Three.js, Canvas và shader; scene dùng thời gian bài hát và dữ liệu phân tích để tính trạng thái hình ảnh. |

Chưa xác minh được prompt Suno gốc hoặc model Suno đã dùng. Không nên suy ra chúng từ tên bản phối.
Theo README, giấy phép MIT áp dụng cho code; nhạc và lời có quyền riêng.

## 2. Tác giả kể gì trong bài “how it was made”?

[Thread của mexicat](https://x.com/_mexicat/status/2103904518929055853) mô tả quá trình:

- Bắt đầu từ ý tưởng còn mơ hồ, rồi tinh chỉnh qua một cuộc trao đổi dài với Claude.
- Đặt yêu cầu thẩm mỹ rõ: typography nổi bật, bảng màu hạn chế, diễn giải lời bằng hình ảnh và lyric đồng bộ chính xác.
- Dùng Whisper và phân tích âm thanh để tìm timing; phần này tốn nhiều công sức.
- Xem và sửa qua hàng chục vòng. Một ví dụ: chuyển chỉ số P(doom) từ HUD cố định vào đồ vật và môi trường của các cảnh.
- Con người giữ vai trò đạo diễn: chọn hướng, đánh giá bản dựng và đưa nhận xét để Claude sửa.

Nguồn cụ thể: [timing](https://x.com/_mexicat/status/2103904533839818855),
[thay đổi P(doom) HUD](https://x.com/_mexicat/status/2103904537048195492),
[vai trò đạo diễn](https://x.com/_mexicat/status/2103904547462853061).

## 3. Pipeline của repo

```text
Audio đã chốt + lời chính xác
  -> tách/phân tích giọng hát, forced alignment và đối chiếu Whisper
  -> timing từng từ + phân tích beat/âm thanh
  -> data/lyrics.json + data/audio.json
  -> storyboard, style bible và timeline cảnh
  -> scene TypeScript / Three.js / Canvas / shader
  -> render chuỗi frame
  -> ffmpeg ghép frame với audio thành video
```

**Forced alignment** tìm đoạn audio tương ứng với từng từ trong phần lời đã biết.
Whisper hỗ trợ đối chiếu; sau đó vẫn có thể cần sửa timing thủ công.
Ví dụ dữ liệu tự đặt, không trích lời của bài gốc:

```json
{ "w": "bay", "start": 2.10, "end": 2.65 }
```

Scene có thể dùng khoảng này để đổi màu, phóng chữ hoặc cho chữ di chuyển.
Dữ liệu beat chỉ cho biết các mốc nhịp; người thiết kế quyết định mỗi mốc kích hoạt hiệu ứng gì.

Các chỗ cần đọc trong repo:

- [`analysis/align.py`](https://github.com/mexicat/pdoom-video/blob/fa164b74fd0b26a99375715b619c55b4a0fb3d05/analysis/align.py): pipeline căn timing lời.
- [`analysis/analyze.py`](https://github.com/mexicat/pdoom-video/blob/fa164b74fd0b26a99375715b619c55b4a0fb3d05/analysis/analyze.py): phân tích âm thanh.
- [`docs/TREATMENT.md`](https://github.com/mexicat/pdoom-video/blob/fa164b74fd0b26a99375715b619c55b4a0fb3d05/docs/TREATMENT.md): brief, phong cách và ý tưởng từng cảnh.
- [`docs/ENGINE.md`](https://github.com/mexicat/pdoom-video/blob/fa164b74fd0b26a99375715b619c55b4a0fb3d05/docs/ENGINE.md): engine, cách triển khai và render.
- [`app/src/timeline.ts`](https://github.com/mexicat/pdoom-video/blob/fa164b74fd0b26a99375715b619c55b4a0fb3d05/app/src/timeline.ts) và [`app/src/scenes/`](https://github.com/mexicat/pdoom-video/tree/fa164b74fd0b26a99375715b619c55b4a0fb3d05/app/src/scenes): timeline và animation thực tế.

## 4. Cách bắt đầu một video mới

Nên làm bản thử **20–30 giây** trước khi dựng cả bài.

1. **Chốt đoạn nhạc và lời chính xác.** Chuẩn bị file audio, đoạn lời tương ứng, tỉ lệ video và phong cách mong muốn. Kiểm tra: audio không đổi sau khi căn timing.
2. **Đánh dấu từng từ và các nhịp chính.** Với đoạn ngắn, có thể làm thủ công trước. Kiểm tra: chữ xuất hiện và highlight khớp giọng hát khi xem lại.
3. **Lên ba cảnh đơn giản.** Ví dụ: highlight theo lời → gõ từng từ → chữ lớn bật ở đoạn nhấn. Chốt font, bảng màu và một motif chung. Kiểm tra: storyboard giải thích rõ từng cảnh và chuyển cảnh.
4. **Viết scene và nối timeline.** Hoàn thành cảnh highlight trước, rồi thêm chuyển động. Kiểm tra: xem được toàn bộ đoạn thử, không thiếu chữ hoặc chồng cảnh sai.
5. **Xuất bản nháp và sửa.** Ưu tiên timing, khả năng đọc chữ và bố cục; sau đó mới render chất lượng cao. Kiểm tra: file video có đúng thời lượng, audio và các mốc đã chọn.

Nếu dùng bài tiếng Việt, timing thủ công cho đoạn thử giúp kiểm chứng sớm việc hiển thị dấu, font và đồng bộ lời.
Pipeline tự động có ích khi mở rộng sang cả bài, nhưng vẫn cần xem lại kết quả.

## 5. Khi tái sử dụng repo

**Thay MP3 và hai file JSON thôi chưa đủ.** Mã nguồn có các lookup dựa trên câu của bài gốc,
các sửa timing thủ công và các mốc section/bar dành cho bài đó.
Phải điều chỉnh timeline, các chỗ tìm lyric và những scene phụ thuộc nội dung hoặc cấu trúc nhạc gốc.

Nên giữ engine và cách render, rồi thiết kế lại dữ liệu, timeline và scene cho bài mới.
Mỗi thay đổi cần phục vụ đoạn video đã chọn; chưa cần thêm hệ thống cấu hình tổng quát.

## 6. Prompt đề xuất để viết storyboard

Đây là prompt mới được đề xuất trong cuộc trao đổi, **không phải prompt gốc của tác giả**:

> Tôi muốn làm lyric video cho đoạn nhạc 30 giây này và phần lời đính kèm.
> Hãy phân tích ý nghĩa lời và các điểm nhấn âm nhạc.
> Đề xuất bảng màu, font và một motif xuất hiện xuyên suốt.
> Viết storyboard với 3–5 cảnh, mỗi cảnh có hành động và chuyển cảnh rõ ràng.
> Chữ phải khớp giọng hát; các chuyển động lớn phải khớp nhịp.
> Trình bày storyboard để tôi góp ý trước khi viết animation.

## 7. Ghi nhận kiểm tra repo ngày 09/10/2026

Các kết quả dưới đây thuộc lần kiểm tra đã thực hiện trên snapshot nêu ở đầu tài liệu:

- TypeScript của ứng dụng và script render: PASS; Vite build: PASS.
- Bun 1.3.14 không đọc được lockfile v2 của repo. Dependency dùng cho kiểm tra được cài bằng `bun install --ignore-scripts --no-save`; lockfile được giữ nguyên.
- `ffmpeg` chưa có trong PATH. Chrome smoke test bị timeout; nguyên nhân chưa được xác minh, kể cả khả năng liên quan đến cờ Metal.
- Chưa xác minh render hoàn chỉnh một video có audio.
- Hai phát hiện từ đọc code: khoảng frame rỗng sau khi làm tròn có thể chờ hoàn tất mãi; lỗi khởi động trước `try/finally` có thể bỏ qua cleanup Chrome/Vite. Chưa tái hiện hai vấn đề này bằng kiểm thử runtime.

Nguồn code cho các phát hiện: [`app/scripts/render.ts`](https://github.com/mexicat/pdoom-video/blob/fa164b74fd0b26a99375715b619c55b4a0fb3d05/app/scripts/render.ts).

## 8. Nguồn và credits

- [Repository và README](https://github.com/mexicat/pdoom-video/tree/fa164b74fd0b26a99375715b619c55b4a0fb3d05).
- [Video của mexicat](https://www.youtube.com/watch?v=5EoO5413dBY).
- [Thread “how it was made”](https://x.com/_mexicat/status/2103904518929055853).
- [Lời và diễn giải của osmarks](https://docs.osmarks.net/hypha/p%28doom%29_song_objectively_correct_interpretation).
- [MusicPerson trên Udio](https://www.udio.com/creators/MusicPerson).
- [Bản nhạc gốc trên YouTube](https://www.youtube.com/watch?v=uEB5E67vcPA).
- [Bản Suno do deckard đăng](https://x.com/slimer48484/status/2097752569212756134).

Tài liệu này không chép lại lời bài hát. Khi làm video riêng, dùng nhạc và lời mà bạn có quyền sử dụng.
