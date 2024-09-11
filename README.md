# GHOST-Auto-Play-BOT-
For Ton Blockchain (Ghost - auto Play BOT) Adjust timings according to your desire.

// READ IT FIRST BEFORE USE //

*🔧 Requirements to Run the Auto-Play BOT:*
1. Install Python: [Download here](https://www.python.org/downloads/)  
2. Install Microsoft Visual C++ Redistributable: [Download here](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist?view=msvc-170)  
3. Coding for Auto-Bot;
   --------------------------------------------------Copy Code under this separator------------------------------------------------

   import ctypes
import random
import time
from pynput.mouse import Controller
import pygetwindow as gw
from pywinauto import findwindows, Application

# Constants for mouse input events
INPUT_MOUSE = 0
MOUSEEVENTF_MOVE = 0x0001
MOUSEEVENTF_ABSOLUTE = 0x8000
MOUSEEVENTF_LEFTDOWN = 0x0002
MOUSEEVENTF_LEFTUP = 0x0004

# Define structures for Windows API input
class MouseInput(ctypes.Structure):
    _fields_ = [("dx", ctypes.c_long), ("dy", ctypes.c_long),
                ("mouseData", ctypes.c_ulong), ("dwFlags", ctypes.c_ulong),
                ("time", ctypes.c_ulong), ("dwExtraInfo", ctypes.POINTER(ctypes.c_ulong))]

class Input_I(ctypes.Union):
    _fields_ = [("mi", MouseInput)]

class Input(ctypes.Structure):
    _fields_ = [("type", ctypes.c_ulong), ("ii", Input_I)]

def click_in_background(x, y):
    """Simulate a mouse click at (x, y) in the background using Windows API."""
    # Convert screen coordinates to absolute coordinates
    screen_width = ctypes.windll.user32.GetSystemMetrics(0)
    screen_height = ctypes.windll.user32.GetSystemMetrics(1)
    abs_x = int(x * 65536 / screen_width)
    abs_y = int(y * 65536 / screen_height)

    # Prepare the input structures
    click_down = Input(type=INPUT_MOUSE, ii=Input_I(mi=MouseInput(dx=abs_x, dy=abs_y,
                             mouseData=0, dwFlags=MOUSEEVENTF_ABSOLUTE | MOUSEEVENTF_MOVE | MOUSEEVENTF_LEFTDOWN,
                             time=0, dwExtraInfo=None)))
    click_up = Input(type=INPUT_MOUSE, ii=Input_I(mi=MouseInput(dx=abs_x, dy=abs_y,
                           mouseData=0, dwFlags=MOUSEEVENTF_ABSOLUTE | MOUSEEVENTF_MOVE | MOUSEEVENTF_LEFTUP,
                           time=0, dwExtraInfo=None)))

    # Send the inputs to the system
    ctypes.windll.user32.SendInput(1, ctypes.byref(click_down), ctypes.sizeof(click_down))
    ctypes.windll.user32.SendInput(1, ctypes.byref(click_up), ctypes.sizeof(click_up))

def random_tapping(center, radius):
    """Perform random clicking within a defined radius around the center."""
    center_x, center_y = center

    # Adjusted time to run the bot
    tap_duration = random.randint(29, 30)  # Random time between 29 to 30 seconds
    total_taps = random.randint(350, 450)  # Random total number of taps

    start_time = time.time()  # Record the start time
    taps_done = 0

    while time.time() - start_time < tap_duration:
        if taps_done >= total_taps:
            break

        # Randomize click position within the defined radius around the center
        random_x = center_x + random.randint(-radius, radius)
        random_y = center_y + random.randint(-radius, radius)

        # Perform the click in the background
        click_in_background(random_x, random_y)
        taps_done += 1

        # Calculate the remaining time
        elapsed_time = time.time() - start_time
        remaining_time = max(0, tap_duration - elapsed_time)
        
        # Adjust the delay dynamically to match the duration and target tap count
        average_delay = remaining_time / max(1, total_taps - taps_done)
        realistic_delay = random.uniform(average_delay * 0.8, average_delay * 1.2)
        time.sleep(realistic_delay)

    print(f"Completed {taps_done} taps in {tap_duration} seconds.")

def select_game_area():
    """Select the area for the bot to click using terminal input."""
    print("Move the mouse to the top-left corner of the game window and press Enter...")
    input()
    mouse = Controller()
    top_left = mouse.position

    print("Move the mouse to the bottom-right corner of the game window and press Enter...")
    input()
    bottom_right = mouse.position

    print(f"Selected area from {top_left} to {bottom_right}.")
    return top_left, bottom_right

def take_a_break():
    """Take a random break between tapping sessions with a countdown."""
    break_duration = random.randint(200, 300)  # 200 to 300 seconds (3.3 to 5 minutes)
    print(f"Taking a break for {break_duration / 60:.2f} minutes.")

    # Countdown timer for the break
    for remaining in range(break_duration, 0, -1):
        print(f"Resuming in {remaining} seconds...", end='\r')
        time.sleep(1)

    print("Break finished! Resuming tapping...")

def make_telegram_on_top():
    """Make the Telegram window stay on top of other windows."""
    try:
        telegram_windows = findwindows.find_elements(title_re="Telegram")
        if telegram_windows:
            app = Application().connect(handle=telegram_windows[0].handle)
            telegram_window = app.window(handle=telegram_windows[0].handle)
            telegram_window.set_focus()
            telegram_window.set_topmost(True)
            print("Telegram window is now set to stay on top.")
        else:
            print("Telegram window not found.")
    except Exception as e:
        print(f"An error occurred: {e}")

if __name__ == "__main__":
    print("Starting the bot...")

    # Make Telegram window stay on top
    make_telegram_on_top()

    # Select the area of the game to tap once
    top_left, bottom_right = select_game_area()

    # Extract coordinates from the tuples
    top_left_x, top_left_y = top_left
    bottom_right_x, bottom_right_y = bottom_right

    # Calculate the center of the selected window
    center_x = (top_left_x + bottom_right_x) // 2
    center_y = (top_left_y + bottom_right_y) // 2
    center = (center_x, center_y)

    # Define a small radius around the center where clicks will happen
    radius = 5  # You can adjust this radius as needed

    while True:
        # Execute the random tapping function
        random_tapping(center, radius)

        # Take a break after each tapping session
        take_a_break()

