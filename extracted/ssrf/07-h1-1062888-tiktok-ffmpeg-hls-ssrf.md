# SSRF and local file read via FFmpeg HLS playlist in uploaded video (TikTok)

Source completeness: full for the technique (the public report body is empty; detail comes from the researcher's summary)

- **Vuln type / sub-variant:** SSRF plus local file read through a server-side video transcoder processing a malicious playlist
- **Target type:** Video upload with server-side conversion (FFmpeg/libavformat)
- **Recon signal:** The site converted uploaded videos on the server. The hunter relied on a long-known FFmpeg playlist trick and the callback user-agent exposed the Lavf version, confirming the converter.
- **Root cause:** FFmpeg's HLS handling follows URLs and local paths listed in a playlist, and the upload pipeline accepted a file (an .avi carrying playlist text) that triggered this behavior during conversion.
- **Exploitation steps:**
  1. Upload an .avi whose content is a minimal HLS playlist listing an attacker URL; receive a callback (SSRF confirmed).
  2. For file read, host a header playlist on the attacker server whose final line is an open URL ending with a question mark, so appended file bytes land in the query string. Ensure the file has no trailing newline (a stray CRLF from the editor initially broke it).
  3. In the uploaded video, use a concat entry joining that header URL with a file:// path; the first line of the target file arrives in the attacker request.
  4. To get the rest, repeat with a subfile entry using start and end offsets and advance the offset per line.
- **Bypasses used:** alternate protocol (file:// via concat/subfile in HLS playlist); no filter evasion mentioned. Note the trailing-newline pitfall on the helper file.
- **Impact:** Outbound requests from the server, arbitrary local file reads (line by line), and a DoS potential the researcher chose not to test. Rated high (7.7).
- **Bounty:** $2,727
- **Lesson / pattern:** Any upload that is transcoded or thumbnailed by FFmpeg should be tested with HLS playlist payloads; verify the callback user-agent to fingerprint the library.
- **Source:**
  - Title: External SSRF and Local File Read via video upload due to vulnerable FFmpeg HLS processing
  - Author: ach
  - URL: https://hackerone.com/reports/1062888
  - Date: 2021-02-15 (reported 2020-12-20)
  - Disclosed report: https://hackerone.com/reports/1062888
  - Raw file: `sources/ssrf/07-h1-1062888-tiktok-ffmpeg-hls-ssrf.md`

Tags: ffmpeg, hls, file-upload, local-file-read, alternate-protocol, oob-callback
