# Speech-to-Text CLI
Creating a Speech-to-Text (STT) Command Line Interface (CLI) in Rust is an exciting project that combines audio processing, machine learning, and CLI development. Lets go through the process of building such a tool.

---

### **1. Understand the Requirements**
Before diving into the code, let's outline the key components of the STT CLI:
- **Audio Input**: Capture audio from a microphone or process an audio file.
- **Speech-to-Text Conversion**: Use a library or service to convert the audio into text.
- **CLI Interface**: Provide a user-friendly command-line interface for interaction.

For this example, we'll use:
- **`cpal`** for audio capture (cross-platform audio library).
- **`hound`** for reading/writing audio files (if needed).
- **`speech-to-text` libraries or APIs** like Google Cloud Speech-to-Text, Mozilla DeepSpeech, or Whisper (via bindings).

---

### **2. Set Up Your Rust Project**
First, create a new Rust project:
```bash
cargo new stt_cli
cd stt_cli
```

Add the necessary dependencies to your `Cargo.toml` file:
```toml
[dependencies]
cpal = "0.15"  # For audio capture
hound = "3.4"  # For audio file handling
reqwest = "0.11"  # For HTTP requests (if using cloud APIs)
serde = { version = "1.0", features = ["derive"] }  # For JSON parsing
tokio = { version = "1", features = ["full"] }  # For async runtime
anyhow = "1.0"  # For error handling
```

---

### **3. Capture Audio Input**
We'll use the `cpal` crate to capture audio from the microphone. Below is an example of how to set up audio capture:

```rust
use cpal::traits::{DeviceTrait, HostTrait, StreamTrait};
use std::sync::{Arc, Mutex};

fn capture_audio() -> Result<Vec<f32>, anyhow::Error> {
    let host = cpal::default_host();
    let device = host.default_input_device().expect("No input device available");

    let config = device.default_input_config()?;
    println!("Capturing audio with config: {:?}", config);

    let sample_rate = config.sample_rate().0;
    let channels = config.channels();

    let audio_data = Arc::new(Mutex::new(Vec::new()));
    let audio_data_clone = Arc::clone(&audio_data);

    let err_fn = move |err| eprintln!("An error occurred on the audio stream: {}", err);

    let stream = device.build_input_stream(
        &config.into(),
        move |data: &[f32], _: &_| {
            let mut audio_data = audio_data_clone.lock().unwrap();
            audio_data.extend_from_slice(data);
        },
        err_fn,
        None,
    )?;

    stream.play()?;

    println!("Recording... Press Ctrl+C to stop.");
    std::thread::sleep(std::time::Duration::from_secs(5)); // Record for 5 seconds

    stream.pause()?;
    Ok(audio_data.lock().unwrap().to_vec())
}
```

This function captures audio for 5 seconds and stores it in a vector of `f32` samples.

---

### **4. Convert Audio to Text**
To convert the captured audio into text, you can use a cloud-based API or a local model. Here’s an example using Google Cloud Speech-to-Text via the `reqwest` crate:

#### **Google Cloud Speech-to-Text Setup**
1. Create a Google Cloud account and enable the Speech-to-Text API.
2. Generate a service account key and download the JSON file.
3. Install the `GOOGLE_APPLICATION_CREDENTIALS` environment variable:
   ```bash
   export GOOGLE_APPLICATION_CREDENTIALS="path/to/your/key.json"
   ```

#### **Send Audio to Google Cloud**
```rust
use reqwest::Client;
use serde_json::json;

async fn transcribe_audio(audio_data: Vec<f32>, sample_rate: u32) -> Result<String, anyhow::Error> {
    let client = Client::new();

    // Encode audio data as base64
    let audio_base64 = base64::encode(audio_data);

    let response = client
        .post("https://speech.googleapis.com/v1/speech:recognize")
        .query(&[("key", "YOUR_API_KEY")])
        .json(&json!({
            "config": {
                "encoding": "LINEAR16",
                "sampleRateHertz": sample_rate,
                "languageCode": "en-US"
            },
            "audio": {
                "content": audio_base64
            }
        }))
        .send()
        .await?;

    let json_response: serde_json::Value = response.json().await?;
    let transcript = json_response["results"][0]["alternatives"][0]["transcript"]
        .as_str()
        .unwrap_or("No transcript found");

    Ok(transcript.to_string())
}
```

---

### **5. Build the CLI**
Now, let's tie everything together into a CLI application. Here's the `main.rs` file:

```rust
#[tokio::main]
async fn main() -> Result<(), anyhow::Error> {
    println!("Starting STT CLI...");

    // Step 1: Capture audio
    let audio_data = capture_audio()?;
    let sample_rate = 44100; // Adjust based on your audio config

    // Step 2: Transcribe audio
    println!("Transcribing audio...");
    let transcript = transcribe_audio(audio_data, sample_rate).await?;

    // Step 3: Display the result
    println!("Transcription: {}", transcript);

    Ok(())
}
```

---

### **6. Run the Application**
Build and run your CLI:
```bash
cargo run
```

When you run the program, it will:
1. Capture audio from the microphone for 5 seconds.
2. Send the audio to Google Cloud Speech-to-Text.
3. Print the transcribed text to the console.

---

### **7. Optional Enhancements**
- **Save Audio to File**: Use the `hound` crate to save the captured audio as a `.wav` file for debugging.
- **Support Multiple Languages**: Allow users to specify the language code via CLI arguments.
- **Local Models**: Integrate with Mozilla DeepSpeech or Whisper for offline transcription.
- **Error Handling**: Improve error handling for network issues, invalid API keys, etc.
