[video crop.py](https://github.com/user-attachments/files/27391399/video.crop.py)
import tkinter as tk
from tkinter import filedialog, messagebox, ttk
import cv2
import os
import threading
from PIL import Image, ImageTk
from moviepy import VideoFileClip

class VideoCropper:
    def __init__(self, root):
        self.root = root
        self.root.title("Video Cropper Pro (Final Fix)")
        self.root.geometry("450x350")
        
        # --- Main UI  ---
        self.main_frame = tk.Frame(root)
        self.main_frame.pack(expand=True, fill="both", padx=20, pady=20)
        
        tk.Label(self.main_frame, text="Video Crop & Fast Render", font=("Arial", 20, "bold")).pack(pady=15)
        
        tk.Label(self.main_frame, text="ชื่อไฟล์ใหม่:").pack()
        self.name_entry = tk.Entry(self.main_frame, width=30, justify='center', font=("Arial", 12))
        self.name_entry.insert(0, "cropped_output")
        self.name_entry.pack(pady=10)

        self.btn_select = tk.Button(self.main_frame, text="Select Video", command=self.select_video, 
                                   bg="#3498db", fg="black", width=25, height=2, font=("Arial", 10, "bold"))
        self.btn_select.pack(pady=20)
        
        self.video_path = ""
        self.roi = [0, 0, 0, 0]
        self.scale_factor = 1.0
        self.rect = None

    def select_video(self):
        self.video_path = filedialog.askopenfilename(filetypes=[("Video files", "*.mp4 *.mov *.avi *.mkv")])
        if self.video_path:
            self.open_crop_window()

    def open_crop_window(self):
        self.crop_win = tk.Toplevel(self.root)
        self.crop_win.title("ลากเมาส์เลือกพื้นที่")
        
        cap = cv2.VideoCapture(self.video_path)
        ret, frame = cap.read()
        cap.release()
        
        if not ret: return
        
        cv2_rgb = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        img_pil = Image.fromarray(cv2_rgb)
        
        # ปรับขนาดภาพตัวอย่าง
        max_size = 800
        w, h = img_pil.size
        self.scale_factor = min(max_size/w, max_size/h) if w > max_size or h > max_size else 1.0
        img_pil = img_pil.resize((int(w*self.scale_factor), int(h*self.scale_factor)), Image.LANCZOS)

        self.photo = ImageTk.PhotoImage(image=img_pil)
        self.canvas = tk.Canvas(self.crop_win, width=self.photo.width(), height=self.photo.height(), cursor="cross")
        self.canvas.pack(padx=10, pady=10)
        self.canvas.create_image(0, 0, anchor="nw", image=self.photo)
        
        btn_frame = tk.Frame(self.crop_win)
        btn_frame.pack(fill="x", pady=10)
        tk.Button(btn_frame, text="Reset", command=self.reset_roi, bg="#f39c12", width=10).pack(side="left", padx=50)
        tk.Button(btn_frame, text="CONFIRM", command=self.confirm_crop, bg="#2ecc71", font=("bold"), width=15).pack(side="left")
        
        self.canvas.bind("<ButtonPress-1>", self.on_press)
        self.canvas.bind("<B1-Motion>", self.on_drag)

    def on_press(self, event):
        self.start_x, self.start_y = event.x, event.y
        if self.rect: self.canvas.delete(self.rect)
        self.rect = self.canvas.create_rectangle(self.start_x, self.start_y, 1, 1, outline='lime', width=3)

    def on_drag(self, event):
        self.canvas.coords(self.rect, self.start_x, self.start_y, event.x, event.y)
        x1, y1 = min(self.start_x, event.x), min(self.start_y, event.y)
        x2, y2 = max(self.start_x, event.x), max(self.start_y, event.y)
        
        w_raw = int((x2-x1)/self.scale_factor)
        h_raw = int((y2-y1)/self.scale_factor)
        
        w_even = w_raw if w_raw % 2 == 0 else w_raw - 1
        h_even = h_raw if h_raw % 2 == 0 else h_raw - 1
        
        self.roi = [int(x1/self.scale_factor), int(y1/self.scale_factor), w_even, h_even]

    def reset_roi(self):
        if self.rect: self.canvas.delete(self.rect)
        self.roi = [0, 0, 0, 0]

    def confirm_crop(self):
        if self.roi[2] < 10: return
        name = self.name_entry.get().strip()
        self.crop_win.destroy()
        threading.Thread(target=self.process_video, args=(self.video_path, self.roi, name), daemon=True).start()

    def process_video(self, path, roi, out_name):
        progress_win = tk.Toplevel(self.root)
        progress_win.title("Rendering")
        progress_win.geometry("300x120")
        tk.Label(progress_win, text="กำลังประมวลผล...\nกรุณาดู % ที่ Terminal ด้านล่าง", pady=10).pack()
        pb = ttk.Progressbar(progress_win, mode='indeterminate', length=200)
        pb.pack(pady=10)
        pb.start()

        try:
            clip = VideoFileClip(path)
            x, y, w, h = roi
            
            # ตัดวิดีโอ (x2, y2 ต้องเป็นพิกัดสิ้นสุด)
            final_clip = clip.cropped(x1=x, y1=y, x2=x+w, y2=y+h)
            
            out_path = os.path.join(os.path.dirname(path), f"{out_name}.mp4")

            # เรนเดอร์แบบแก้ปัญหา QuickTime และแก้ปัญหาเลขคี่
            final_clip.write_videofile(
                out_path, 
                codec="libx264", 
                audio_codec="aac",
                preset="ultrafast",
                ffmpeg_params=["-pix_fmt", "yuv420p"] 
            )
            
            clip.close()
            final_clip.close()
            progress_win.destroy()
            
            messagebox.showinfo("Success", f"เสร็จเรียบร้อย!\nไฟล์อยู่ที่: {out_path}")
            os.system(f'open "{os.path.dirname(out_path)}"') 
            self.root.quit()

        except Exception as e:
            if progress_win.winfo_exists(): progress_win.destroy()
            messagebox.showerror("Error", f"Error: {str(e)}")

if __name__ == "__main__":
    root = tk.Tk()
    app = VideoCropper(root)
    root.mainloop()
