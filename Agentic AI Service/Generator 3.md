# This Rust program demonstrates how to use the **HlsKit** crate to process a video file (in this case, `sample.mp4`) 
and convert it into an HLS-compatible format with multiple resolutions for adaptive streaming. Here's a simplified explanation of what the code does:

---

### **Step-by-Step Explanation**

1. **Import Necessary Modules:**
   ```rust
   use std::{env, fs::File, io::Read};
   use hlskit::{
       models::hls_video_processing_settings::{FfmpegVideoProcessingPreset, HlsVideoProcessingSettings},
       process_video,
   };
   ```
   - The program imports standard Rust libraries (`std`) for file handling and error management.
   - It also imports specific components from the `hlskit` crate:
     - `HlsVideoProcessingSettings`: Defines settings for processing each video resolution.
     - `FfmpegVideoProcessingPreset`: Specifies the encoding speed/quality trade-off.
     - `process_video`: The main function that handles video conversion.

2. **Set Up the Asynchronous Main Function:**
   ```rust
   #[tokio::main]
   async fn main() -> Result<(), Box<dyn std::error::Error>> {
   ```
   - The `#[tokio::main]` macro makes the program asynchronous, allowing non-blocking operations like video processing.
   - The `Result` return type ensures that errors are handled gracefully.

3. **Read the Input Video File:**
   ```rust
   let mut buf = Vec::new();
   File::open("src/sample.mp4")?.read_to_end(&mut buf)?;
   ```
   - Opens the `sample.mp4` file and reads its contents into a byte buffer (`buf`).
   - This buffer will be passed to the `process_video` function for processing.

4. **Define Video Processing Settings:**
   ```rust
   let result = process_video(
       buf,
       vec![
           HlsVideoProcessingSettings {
               resolution: (1920, 1080),
               constant_rate_factor: 28,
               preset: FfmpegVideoProcessingPreset::Fast,
           },
           HlsVideoProcessingSettings {
               resolution: (1280, 720),
               constant_rate_factor: 28,
               preset: FfmpegVideoProcessingPreset::Fast,
           },
           HlsVideoProcessingSettings {
               resolution: (854, 480),
               constant_rate_factor: 28,
               preset: FfmpegVideoProcessingPreset::Fast,
           },
       ],
   )
   .await?;
   ```
   - Calls the `process_video` function with:
     - The video data (`buf`).
     - A list of processing settings for different resolutions:
       - **1080p (1920x1080):** High-quality video.
       - **720p (1280x720):** Medium-quality video.
       - **480p (854x480):** Lower-quality video for slower connections.
   - Each setting includes:
     - `resolution`: The width and height of the video.
     - `constant_rate_factor`: Controls video quality (lower values mean better quality but larger files).
     - `preset`: Determines the encoding speed/quality trade-off (`Fast` means faster encoding but slightly lower quality).

5. **Print the Master Playlist:**
   ```rust
   println!("Master playlist:\n{}", String::from_utf8_lossy(&result.master_m3u8_data));
   ```
   - After processing, the `process_video` function returns a result containing the **master playlist** (an `.m3u8` file).
   - The master playlist lists all the available video resolutions and their corresponding `.ts` (video segment) files.
   - This playlist is printed to the console for inspection.

6. **Handle Errors and Exit:**
   ```rust
   Ok(())
   ```
   - If everything succeeds, the program exits without errors.

---

### **What Does This Program Do in Simple Terms?**

1. **Input:** It takes a video file (`sample.mp4`) as input.
2. **Processing:** It converts the video into multiple resolutions (1080p, 720p, 480p) using FFmpeg.
3. **Output:** It generates an HLS-compatible output, including:
   - Multiple `.ts` files (small video chunks for each resolution).
   - A master playlist (`.m3u8`) that tells video players which resolution to use based on network conditions.
4. **Display:** It prints the master playlist to the console so you can see how the video is structured for streaming.

---

### **Why Is This Useful?**

- **Adaptive Streaming:** By generating multiple resolutions, the video can adapt to the viewer's internet speed. For example:
  - Fast connections get 1080p.
  - Slower connections get 480p.
- **Scalability:** HLS is widely supported by browsers, mobile devices, and streaming platforms.
- **Performance:** Using Rust and asynchronous processing ensures the program can handle large video files efficiently.

---

### **Key Takeaways**

- This program uses the **HlsKit** crate to simplify HLS video processing.
- It demonstrates how to convert a single video into multiple resolutions for adaptive streaming.
- The output includes a master playlist (`.m3u8`) that enables smooth playback across devices.

If you're building a video streaming application, this example shows how to integrate HLS processing into your Rust project!
