# 🚗 Real-Time Road Lane Detection (OpenCV) — Colab Ready

This repository contains a single Colab notebook: `Real_Time_Road_Lane_Detection.ipynb` plus this `README.md`.

It performs **classical (non-deep-learning) lane detection** on a road video using:

- **Edge Detection** (Gaussian blur + **Canny**)
- **Region of Interest (ROI)** masking (polygon mask to focus on the road)
- **Probabilistic Hough Transform** (**`cv2.HoughLinesP`**)
- **Left/Right lane estimation** via **slope/intercept + weighted averaging**
- **Lane overlay** on top of each frame to produce an output video


---

<details>
<summary><strong>🔗 Colab link (open this notebook)</strong></summary>

Open directly here:

[https://colab.research.google.com/drive/14kYHoFYE_Raqk_9n9VGWLU97kG-iScPN?usp=sharing](https://colab.research.google.com/drive/14kYHoFYE_Raqk_9n9VGWLU97kG-iScPN?usp=sharing)

Then click **“Open in Colab”** if prompted.

</details>

---

## ✨ Quick Start (fastest path)

<details>
<summary><strong>1) Open in Colab</strong></summary>

Use the Colab link above.

Then ensure you have a runtime connected:

- `Runtime` -> `Run all` (or `Connect`) depending on Colab UI.

</details>

<details>
<summary><strong>2) Upload your video to Colab</strong></summary>

In Colab, upload through the left sidebar:

1. Click **Files**
2. Click **Upload**
3. Select your video (example: `my_road.mp4`)

Uploaded files land in:

- `/content/` (Colab working directory)

</details>

<details>
<summary><strong>3) Change the input path (the most important step)</strong></summary>

In the last part of the notebook, the driver call looks like this (from the notebook logic):

- `process_video('/content/test2.mp4', 'output.mp4')`

You must either:

1. Upload a file named **`test2.mp4`**, or
2. Change `'/content/test2.mp4'` to your uploaded filename

Example:

- If you uploaded `highway.mp4`, use:
  - `process_video('/content/highway.mp4', 'output.mp4')`

</details>

<details>
<summary><strong>4) Run the notebook</strong></summary>

Use:

- **Runtime** -> **Run all**

The notebook will produce:

- `output.mp4` (with detected lane overlays)

and preview it inline.

</details>

---

## 🎯 What you’ll get (output)

For each frame, the notebook draws the estimated lane boundaries as **thick colored lines** (red by default) and writes a processed video.

### Expected visual result

- Red lines representing:
  - **left lane** (negative slope segments)
  - **right lane** (positive slope segments)
- Lines are formed by:
  - averaging many Hough segments into a stable left/right lane model

---

## 🧠 How the algorithm works (high detail)

This notebook is a classical computer vision pipeline. Here’s what each stage does, aligned with GeeksforGeeks:

### 1) Read video and process frame-by-frame

The notebook uses `moviepy` to:

- open the video file
- iterate frames
- apply `frame_processor(frame)` to each frame
- write an output mp4

The driver uses a subclip and resizing for speed (see “Video settings” below).

### 2) Grayscale conversion

Frames come in BGR format (OpenCV convention). The notebook converts to grayscale because processing a single channel is faster.

### 3) Noise reduction (Gaussian blur)

Gaussian blur reduces false edges caused by noise. This makes Canny output more stable.

### 4) Edge detection (Canny)

The Canny edge detector traces strong intensity changes after blur.

Notebook logic (typical values from your code):

- `cv2.Canny(blur, 50, 150)`

Tuning `50/150` strongly affects results:

- too low: noisy edges, many incorrect lines
- too high: missing edges, lanes won’t be detected

### 5) Region of Interest (ROI) masking

Instead of running Hough on the entire image, the notebook focuses on a trapezoid-like polygon where the road lanes usually appear.

GeeksforGeeks describes ROI masking as:

- create a mask of the same size as the edge image
- fill a polygon corresponding to the lane region
- apply `bitwise AND` so edges outside the polygon are ignored

Your notebook uses a polygon defined by relative coordinates:

- bottom area: wider
- top area: narrower (lanes converge)

### 6) Probabilistic Hough Transform

The notebook calls `cv2.HoughLinesP` to detect straight line segments from the masked edge image.

Notebook logic:

- `cv2.HoughLinesP(image, rho=1, theta=np.pi/180, threshold=20, minLineLength=20, maxLineGap=500)`

Interpretation of parameters:

- `threshold`: minimum votes to accept a line
- `minLineLength`: reject very short segments
- `maxLineGap`: allow gaps between points so segments can be linked

### 7) Left/right lane estimation via slope & intercept

Hough returns many line segments (more than 2). To get a stable lane boundary, the notebook:

1. Computes slope `m` and intercept `b` for each segment:
   - `m = (y2 - y1) / (x2 - x1)`
   - `b = y1 - m*x1`
2. Separates segments into:
   - **left lane** if `m < 0` (negative slope)
   - **right lane** if `m > 0` (positive slope)
3. Computes a **weighted average** using segment lengths as weights:
   - longer segments influence the averaged lane more

This matches the GeeksforGeeks “Average_Slope_Intercept” explanation.

### 8) Convert the averaged model back into pixel endpoints

Using two y-values (bottom of frame and a second y at ~60% height), it computes x positions for each lane model and returns endpoints suitable for drawing.

In your code:

- `y1 = image.shape[0]`
- `y2 = int(y1 * 0.6)`

### 9) Draw lanes and blend with the frame

The notebook draws lane lines on a blank image, then overlays it:

- `cv2.line(...)` into `line_image`
- `cv2.addWeighted(original, 1.0, line_image, 1.0, 0.0)`

Output is a clear visualization of detected lanes.

---

## 📦 Video settings in this notebook (what you may need to edit)

The notebook’s pipeline includes performance-oriented settings inside the video driver.

### Subclip time range

The driver uses a subclip (your notebook logic includes):

- `.subclip(0, 10)`

Meaning:

- it processes only the first **10 seconds**

To process more:

- change `0, 10` to something like `0, 30`

### Resize for speed

It also resizes frames:

- `.resize(0.5)`

Meaning:

- halves the resolution to speed up processing

If lanes are too thin or detection is unstable, try:

- `resize(0.75)` or `resize(1.0)` (slower but often better)

### Output FPS and audio

The notebook writes:

- `fps=24`
- `audio=False`

---

## 📤 Where to upload the video in Colab

### Option A (recommended): Files sidebar upload

1. On the left, open **Files**
2. Click **Upload**
3. Choose your `.mp4` (or `.avi` if supported)

Your file will appear in:

- `/content/your_video_name.mp4`

### Option B: Upload via code

Run (if your notebook doesn’t already include it):

```python
from google.colab import files
uploaded = files.upload()
```

After upload, confirm the filename is present in `/content/`.

---

## 🛠️ Change the video path (exact instruction)

In the notebook, look for the driver call that runs lane detection:

```python
process_video('/content/test2.mp4', 'output.mp4')
```

You must change `'/content/test2.mp4'` to match the uploaded filename.

### Common mistakes

- Uploading `my_video.mp4` but leaving `test2.mp4` in code
- Uploading to a different folder (everything uploaded from Colab UI is in `/content/` by default)
- Using the wrong extension (`.mp4` vs `.avi`)

### Example mapping

- Upload file: `highway.mp4`
- Code: `process_video('/content/highway.mp4', 'output.mp4')`

---

## ✅ How to run locally (optional)

If you want to run outside Colab:

1. Install dependencies:
   - `pip install opencv-python moviepy numpy`
2. Download the `.py` generated from the notebook or run the notebook directly
3. Update paths:
   - replace `'/content/...'` with your local path

Example idea:

- local video path: `C:/videos/highway.mp4`
- output: `output.mp4`

---

## 🔧 Parameter tuning guide (make it work better)

Lane detection varies a lot depending on camera angle, road markings, lighting, and resolution.

Use these knobs first:

<details>
<summary><strong>ROI tuning (polygon shape)</strong></summary>

ROI is defined by four points based on image width/height:

- `bottom_left  = [cols * 0.1, rows * 0.95]`
- `top_left     = [cols * 0.4, rows * 0.6]`
- `top_right    = [cols * 0.6, rows * 0.6]`
- `bottom_right = [cols * 0.9, rows * 0.95]`

If your road occupies a different region:

- adjust these ratios so the trapezoid tightly covers lane lines

Rule of thumb:

- Move `top_left/top_right` outward if lanes aren’t included.
- Move `bottom_left/bottom_right` inward if you pick up edges from shoulders/curbs.

</details>

<details>
<summary><strong>Canny thresholds</strong></summary>

Notebook uses:

- `cv2.Canny(blur, 50, 150)`

If detection is too noisy:

- increase thresholds (e.g. `70, 200`)

If detection misses lanes:

- decrease thresholds (e.g. `30, 100`)

</details>

<details>
<summary><strong>Hough Transform parameters</strong></summary>

Notebook uses:

- `threshold=20`
- `minLineLength=20`
- `maxLineGap=500`

If you see many random lines:

- raise `threshold`
- raise `minLineLength`

If lane segments are broken:

- increase `maxLineGap`

</details>

<details>
<summary><strong>Frame stabilization</strong></summary>

This notebook already improves stability by:

- separating left/right by slope sign
- computing weighted average with segment lengths

For even smoother results:

- increase `minLineLength`
- consider processing a larger resolution (reduce `resize` down to 0.5 only if needed)

</details>

---

## 🧪 Troubleshooting (very common issues)

<details>
<summary><strong>No red lines / blank output video</strong></summary>

Possible causes:

- ROI polygon doesn’t include the actual lane area
- Canny thresholds are too high (edges aren’t detected)
- Hough settings are too strict

Fix order:

1. Adjust ROI vertices to cover the lanes
2. Lower Canny thresholds slightly
3. Lower Hough `threshold` or `minLineLength`

</details>

<details>
<summary><strong>Wrong direction (left vs right lanes swapped)</strong></summary>

The notebook assumes:

- negative slope => left lane
- positive slope => right lane

If your camera orientation is unusual (or the road appears flipped):

- you may need to swap the slope condition logic

</details>

<details>
<summary><strong>Flickering / unstable lane lines</strong></summary>

Possible causes:

- too much noise in edges (Canny tuning)
- short Hough segments dominating averages

Fix order:

1. Increase `minLineLength`
2. Increase blur kernel size (e.g. `(7, 7)` instead of `(5, 5)`) if needed
3. Adjust ROI to remove distracting edges

</details>

<details>
<summary><strong>Very slow processing</strong></summary>

If it’s slow:

- keep `resize(0.5)`
- reduce the subclip length (e.g. `subclip(0, 5)`)
- avoid processing very high-resolution videos

</details>

---

## 🧾 Implementation map (where each step lives)

Inside the notebook, the key functions are organized like this:

- `region_selection(image)`:
  - creates the ROI polygon mask and applies bitwise AND
- `hough_transform(image)`:
  - calls `cv2.HoughLinesP`
- `average_slope_intercept(lines)`:
  - computes weighted average left/right lane line models
- `pixel_points(y1, y2, line)`:
  - converts (slope, intercept) into drawable endpoints
- `lane_lines(image, lines)`:
  - builds final left/right lane endpoints
- `draw_lane_lines(image, lines, ...)`:
  - draws lanes onto a blank overlay and blends with frame
- `frame_processor(image)`:
  - full pipeline per frame (with `try/except` to prevent crashes)
- `process_video(input_path, output_path)`:
  - loads video, applies `fl_image(frame_processor)`, writes output mp4

---

## 📚 Credits

- GeeksforGeeks article:
  - https://www.geeksforgeeks.org/machine-learning/opencv-real-time-road-lane-detection/
- OpenCV + MoviePy + NumPy (used by the notebook)

---

## 📌 Notes / Limitations (important)

This notebook is designed for classical lane detection. It works best when:

- lane markings are relatively clear
- road view resembles the ROI perspective assumption (trapezoid lanes)
- lanes are mostly straight (or mildly curved)

Limitations:

- curves and complex road geometry can reduce accuracy (because averaging assumes near-straight segments)
- steep/odd camera angles may break ROI coverage
- heavy rain/fog/shadows can cause noisy edges

---

## ❓ FAQ

<details>
<summary><strong>Can I use a webcam?</strong></summary>

Not directly in the current notebook. This code is video-file driven (`VideoFileClip`).

You’d need a streaming capture loop and frame-by-frame processing.

</details>

<details>
<summary><strong>Does it require GPU?</strong></summary>

No. It’s CPU-based classical CV. GPU isn’t required.

</details>

---

## 📎 Sample resources (from the reference article)

GeeksforGeeks also mentions a sample road dataset:

- Dataset: [https://github.com/rslim087a/road-video](https://github.com/rslim087a/road-video)

If you’re trying to validate the notebook quickly, downloading a sample `mp4` from that dataset is a good way to confirm your Colab environment works end-to-end.

---

## 🧰 Dependencies & what they do

Your notebook installs/uses:

- `opencv-python` (`cv2`)
  - image processing, Canny edges, Hough line detection, drawing overlays
- `numpy`
  - numeric operations for ROI masking and lane averaging
- `moviepy`
  - reading a video and writing an output `mp4`
- (inside notebook) `IPython.display` + `base64`
  - embeds the resulting `output.mp4` directly in the notebook output cell

If you hit import errors in Colab, re-run the dependency/install cell before running the processing cell.

---

## 🧪 How to debug if lane detection fails

If the notebook runs but the output has no red lanes or looks wrong, debug from the inside out:

<details>
<summary><strong>Step 1: Confirm the video path is correct</strong></summary>

Make sure the driver call uses a file that actually exists in Colab:

- verify the uploaded filename in `/content/`
- update `process_video('/content/<your_file>.mp4', 'output.mp4')`

If you get a file-not-found error, stop here and fix the path first.

</details>

<details>
<summary><strong>Step 2: Verify edges (Canny) look reasonable</strong></summary>

In the notebook’s `frame_processor`, the relevant line is:

- `edges = cv2.Canny(blur, 50, 150)`

If edges are too noisy:

- increase Canny thresholds

If edges are missing:

- decrease Canny thresholds

Suggested experiment:

- Try `cv2.Canny(blur, 30, 100)` first if nothing shows up

</details>

<details>
<summary><strong>Step 3: Verify ROI mask coverage</strong></summary>

The ROI is generated by a polygon using these ratios:

- `bottom_left  = [cols * 0.1, rows * 0.95]`
- `top_left     = [cols * 0.4, rows * 0.6]`
- `top_right    = [cols * 0.6, rows * 0.6]`
- `bottom_right = [cols * 0.9, rows * 0.95]`

If your road is in a different place, detection will fail even if Canny is good.

Quick fix:

- move `top_left/top_right` outward if lanes are not included
- move `bottom_left/bottom_right` inward if you capture too much background

</details>

<details>
<summary><strong>Step 4: Verify Hough lines exist</strong></summary>

The Hough call is:

- `cv2.HoughLinesP(image, rho=1, theta=np.pi/180, threshold=20, minLineLength=20, maxLineGap=500)`

If `hough is None` frequently:

- reduce `threshold`

If you see lots of random lines:

- increase `threshold`
- increase `minLineLength`

</details>

<details>
<summary><strong>Step 5: Understand left/right separation</strong></summary>

The notebook assumes:

- negative slope segments are **left**
- positive slope segments are **right**

If your camera view makes the road appear “flipped”, this rule might need swapping.

</details>

---

## 🧮 Lane averaging (why it looks stable)

Hough returns many small segments. Directly drawing every segment creates noisy lane overlays.

Instead, the notebook:

1. Computes each segment’s slope and intercept (`m`, `b`)
2. Filters left/right using sign of slope
3. Uses a **weighted average**, where each segment’s weight is its pixel length

That’s why you end up with just two stable lane boundaries: one averaged left lane and one averaged right lane.

This is the same “Average_Slope_Intercept” idea explained in the GeeksforGeeks article.

---

## 🎬 Output explanation (how the video preview is embedded)

After processing, the notebook:

1. Writes `output.mp4`
2. Reads it as bytes
3. Converts it to a Base64 `data:` URL
4. Displays it using HTML:

- `<video width=500 controls> ... </video>`

So you don’t need to download the file manually unless you want to.

---

## 🧷 Parameter cheat-sheet (fast tuning)

| Parameter | Used for | If detection is weak | If output is noisy |
|---|---|---|---|
| Canny low/high (e.g. `50,150`) | Edge quality | decrease both | increase both |
| `threshold` in `HoughLinesP` | line acceptance | decrease | increase |
| `minLineLength` | reject tiny segments | decrease slightly | increase |
| `maxLineGap` | bridge nearby segments | increase (if broken lanes) | keep moderate/high only if lanes are segmented |
| ROI polygon points | where lines can be found | expand ROI to cover lanes | shrink ROI to exclude background |
| `resize(0.5)` | performance vs detail | use higher resize (e.g. `0.75`) | keep lower resize |
| `subclip(0,10)` | processing time | extend (e.g. `0,30`) | shorten for quick tests |

---
