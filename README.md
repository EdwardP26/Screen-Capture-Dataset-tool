# Screen Capture and Dataset Generation Tool

## Project Description

This project provides a tool for capturing screenshots from a specific window on your desktop and saving them as images in a designated folder. 

These images can be used for various purposes, such as:
- **Object Detection Training**: Use the captured images to train models like YOLOv4 Tiny for object detection.
- **Game Data Collection**: Capture game screens for building datasets to analyze in-game events or for training AI models.
- **Application Monitoring**: Automate the process of recording screenshots from a running application for monitoring or troubleshooting.

The tool works by taking continuous screenshots from the specified window and saving them into the `images` folder, which can then be used for dataset creation or analysis.

---
## Setup Instructions

1. **Open the Application or Window** you want to capture.
2. **Get the Window Title Name**:
    - Open the application and note the title from the window's title bar.
    - Alternatively, you can use Task Manager (press Ctrl + Shift + Esc) to find the window title. In Task Manager, find the application under the Processes tab, and note its name.
    - Replace the `window_name` variable in the code with the title of the desired window.
3. **Run the Code** to start capturing images.
    - Run the following command in your terminal to install the necessary libraries: pip install numpy pywin32 pillow.
---
## 1. Import Necessary Libraries
Make sure to do: pip install numpy pywin32 pillow
```python
import numpy as np
import win32gui, win32ui, win32con
from PIL import Image
from time import sleep
import os
```
## 2. Define the Window Capture Class
This class will capture the screenshots from the window you specify by its title.
```python
class WindowCapture:
    w = 0
    h = 0
    hwnd = None

    def __init__(self, window_name):
        # Find the window handle by the given window name
        self.hwnd = win32gui.FindWindow(None, window_name)
        if not self.hwnd:
            raise Exception('Window not found: {}'.format(window_name))

        # Get the window dimensions
        window_rect = win32gui.GetWindowRect(self.hwnd)
        self.w = window_rect[2] - window_rect[0]
        self.h = window_rect[3] - window_rect[1]

        # Adjust for window borders and title bar
        border_pixels = 8
        titlebar_pixels = 30
        self.w = self.w - (border_pixels * 2)
        self.h = self.h - titlebar_pixels - border_pixels
        self.cropped_x = border_pixels
        self.cropped_y = titlebar_pixels
```
## 3. Define the Screenshot Function
This function captures a screenshot from the specified window.
```python
    def get_screenshot(self):
        # Get the device context (DC) of the window
        wDC = win32gui.GetWindowDC(self.hwnd)
        dcObj = win32ui.CreateDCFromHandle(wDC)
        cDC = dcObj.CreateCompatibleDC()
        dataBitMap = win32ui.CreateBitmap()
        dataBitMap.CreateCompatibleBitmap(dcObj, self.w, self.h)
        cDC.SelectObject(dataBitMap)

        # Copy the window image to the bitmap
        cDC.BitBlt((0, 0), (self.w, self.h), dcObj, (self.cropped_x, self.cropped_y), win32con.SRCCOPY)

        # Save the screenshot in memory
        signedIntsArray = dataBitMap.GetBitmapBits(True)
        img = np.frombuffer(signedIntsArray, dtype='uint8')
        img.shape = (self.h, self.w, 4)

        # Free resources
        dcObj.DeleteDC()
        cDC.DeleteDC()
        win32gui.ReleaseDC(self.hwnd, wDC)
        win32gui.DeleteObject(dataBitMap.GetHandle())

        # Convert BGR to RGB and return the image as a numpy array
        img = img[..., :3]
        img = np.ascontiguousarray(img)

        return img
```
## 4. Define the Function to Generate the Dataset
This function captures a specified number of images from the window as well as intervals and saves them into a folder.
```python
def generate_image_dataset(self):
        # Create 'images' folder if it does not exist
        if not os.path.exists("images"):
            os.mkdir("images")
        img_counter = len(os.listdir("images"))
        while True:
            img = self.get_screenshot()
            img_path = f"./images/img_{img_counter}.jpg"
            im = Image.fromarray(img[..., [2, 1, 0]])  # Convert BGR to RGB
            im.save(img_path)

            # Print the image number and location to the terminal
            print(f"Saved: {img_path}")

            img_counter += 1
            sleep(1)  # Set delay between screenshots (seconds)
```
## 5. Update the window_name Variable
Replace the window_name with the title of the window you want to capture.
```python
window_name = "WindowNameHere"  # Replace this with the window title
wincap = WindowCapture(window_name)
```
## 6. Run the Dataset Generation
Starts capturing images
```python
wincap.generate_image_dataset()
```
---
## Note:
  - You can modify the delay parameter in the generate_image_dataset function to control the interval between each screenshot. (Step)
  - Example: sleep = 2 (Screenshot every 2 seconds).
