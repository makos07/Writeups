
<h1>Security Footage</h1>

Perform digital forensics on a network capture to recover footage from a camera.

"Someone broke into our office last night, but they destroyed the hard drives with the security footage. Can you recover the footage?"

Download task file:
security-footage-1648933966395.pcap

It's a pcap file.
Opened it with wireshark:

![1](https://github.com/user-attachments/assets/3b96a15a-df12-4354-b6ac-63dbbfc33eac)

Follow the TCP stream
![2025-06-27_11h25_19](https://github.com/user-attachments/assets/3f22c944-7452-4a8c-b75f-da7aa4d85220)

There are multiple "Content-type: image/jpeg" rows in the stream. So it contains multiple images in jpeg format.

Select the stream from the server, and show as Raw data:
![Selection_002](https://github.com/user-attachments/assets/03bcd366-02f8-48f0-9b5a-8a9bcb9cf3c2)

Saved the file as "raw_data_dump"
Extract all images from the raw data with this python script:
```python
import re
import os

def extract_jpegs(input_path, output_dir):
    with open(input_path, 'rb') as f:
        data = f.read()

    # Find all JPEGs using SOI and EOI markers
    jpeg_pattern = re.compile(b'\xff\xd8.*?\xff\xd9', re.DOTALL)
    matches = jpeg_pattern.findall(data)

    print(f"Found {len(matches)} JPEG image(s).")

    os.makedirs(output_dir, exist_ok=True)

    for i, jpeg in enumerate(matches):
        with open(f"{output_dir}/frame_{i:03}.jpg", 'wb') as out_file:
            out_file.write(jpeg)

    print("Extraction complete.")

# Example usage
extract_jpegs("raw_data_dump", "output_jpegs")
```
Now we got the images of a screen:
![frame_000](https://github.com/user-attachments/assets/fac8f9b8-7f9d-4b43-93cf-1045dfffc114)

Let's merge these images to a video, for easier reading:
```bash
ffmpeg -framerate 10 -i output_jpegs/frame_%03d.jpg -c:v libx264 -pix_fmt yuv420p output_video.mp4
```
An we can read the flag:
flag{5ebf...}
