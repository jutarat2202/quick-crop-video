# quick-crop-video
Fast and simple video cropping tool with drag-to-select preview and MP4 export.

[README.md](https://github.com/user-attachments/files/27391379/README.md)
# Video Cropper Pro

A lightweight Python desktop app for visually cropping videos and exporting QuickTime-compatible MP4 files.

## Features

- Select a video file from your computer
- Preview the first frame before cropping
- Drag to select the crop area
- Reset and reselect the crop area
- Export the cropped video as an MP4 file
- Uses H.264 video codec and AAC audio codec
- Exports with `yuv420p` pixel format for better QuickTime compatibility
- Automatically adjusts crop dimensions to even numbers to avoid FFmpeg encoding errors

## Tech Stack

- Python
- Tkinter
- OpenCV
- Pillow
- MoviePy
- FFmpeg

## Requirements

Install Python 3.9 or newer.

Install the required Python packages:

```bash
pip install opencv-python pillow moviepy
```

On some Linux systems, you may also need Tkinter:

```bash
sudo apt-get install python3-tk
```

## Usage

Run the app:

```bash
python "video crop.py"
```

Then:

1. Enter the output file name.
2. Click **Select Video**.
3. Choose a video file.
4. Drag over the preview to select the crop area.
5. Click **CONFIRM**.
6. Wait for the video to finish rendering.

The cropped video will be saved in the same folder as the original video.

## Supported Input Formats

The app currently supports:

- `.mp4`
- `.mov`
- `.avi`
- `.mkv`

## Output

The app exports the cropped video as:

```text
your_output_name.mp4
```

The output file uses:

- Video codec: `libx264`
- Audio codec: `aac`
- Pixel format: `yuv420p`

## Notes

The crop width and height are automatically adjusted to even numbers. This prevents common FFmpeg errors such as:

```text
width not divisible by 2
```

## Project Structure

```text
.
├── video crop.py
└── README.md
```

## License

This project is open source. You can add your preferred license here.
