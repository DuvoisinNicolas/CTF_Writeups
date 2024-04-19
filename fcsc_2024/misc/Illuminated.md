
Here's the script i generated using chatgpt mostly:
```python
from scapy.all import *
from PIL import Image, ImageSequence


def merge_gifs(gif1_path, gif2_path, output_path):
    gif1 = Image.open(gif1_path)
    gif2 = Image.open(gif2_path)
    
    merged_frames = []
    for frame1, frame2 in zip(ImageSequence.Iterator(gif1), ImageSequence.Iterator(gif2)):
        merged_frame = Image.new("RGB", frame1.size)
        pixels1 = frame1.load()
        pixels2 = frame2.load()
        merged_pixels = merged_frame.load()
        
        for x in range(merged_frame.width):
            for y in range(merged_frame.height):
                color1 = pixels1[x, y]
                color2 = pixels2[x, y]
                if color1 != (0, 0, 0):
                    merged_pixels[x, y] = color1
                else:
                    merged_pixels[x, y] = color2
        
        merged_frames.append(merged_frame)
    
    merged_frames[0].save(output_path, save_all=True, append_images=merged_frames[1:], duration=20, loop=0)

def extract_dmx_data(packet):
    if packet.haslayer(Raw):
        raw_data = packet[Raw].load
        universe = int.from_bytes(raw_data[14:16], byteorder='little')
        dmx_data = list(raw_data[18:530])
        return universe, dmx_data

def split_into_rgb(dmx_data):
    rgb_data = []
    for i in range(0, len(dmx_data) - 2, 3):
        rgb_data.append(dmx_data[i:i+3])
    if len(rgb_data[-1]) < 3:
        rgb_data[-1].append(0)
    return rgb_data

def get_packet_data(file_path, universe):
    packets = rdpcap(file_path)
    packet_data = []
    for packet in packets:
        packet_universe, dmx_data = extract_dmx_data(packet)
        if packet_universe == universe and dmx_data is not None:
            rgb_data = split_into_rgb(dmx_data)
            packet_data.append(rgb_data)
    return packet_data

def create_images(packet_data, start_row=0):
    images = []
    for rgb_data in packet_data:
        width = 16
        height = 16
        image = Image.new("RGB", (width, height))
        pixels = image.load()
        row_index = 0
        direction = 1  # 1 for left to right, -1 for right to left
        for y in range(start_row, height):
            for x in range(width):
                index = row_index * width + x
                if index < len(rgb_data):  # Ensure index is within rgb_data bounds
                    if direction == 1:
                        pixels[x, y] = tuple(rgb_data[index])
                    else:
                        pixels[width - 1 - x, y] = tuple(rgb_data[index])
                else:
                    pixels[x, y] = (0, 0, 0)  # Set to black if index out of bounds
            # Toggle direction and update row index
            direction *= -1
            row_index += 1
        images.append(image)
    return images

def save_gif(images, file_name, duration=50):
    images[0].save(file_name, save_all=True, append_images=images[1:], duration=duration, loop=0)

if __name__ == "__main__":
    file_path = "Illuminated/capture.pcap"
    
    # Process data for universe 0
    universe_0 = 0
    start_row_1 = 0
    packet_data_0 = get_packet_data(file_path, universe_0)
    print(len(packet_data_0))
    if packet_data_0:
        images_0 = create_images(packet_data_0)
        save_gif(images_0, "universe_0.gif", duration=20)  # Set duration to 100ms for slower speed
        print("GIF for Universe 0 created successfully.")
    else:
        print(f"No packets found in universe {universe_0} or data extraction failed.")
    
    # Process data for universe 1 with offset
    universe_1 = 1
    start_row_1 = 10  # Offset for universe 1
    packet_data_1 = get_packet_data(file_path, universe_1)
    print(len(packet_data_1))
    if packet_data_1:
        images_1 = create_images(packet_data_1, start_row_1)
        save_gif(images_1, "universe_1.gif", duration=20)  # Set duration to 100ms for slower speed
        print("GIF for Universe 1 created successfully.")
    else:
        print(f"No packets found in universe {universe_1} or data extraction failed.")

    
    gif1_path = "universe_1.gif"
    gif2_path = "universe_0.gif"
    output_path = "merged.gif"
    
    merge_gifs(gif1_path, gif2_path, output_path)


```

The "merge gif" part doesn't work quite well, i feel like there are less frames in universe 1 than in universe 0.
So i just rebuilt the flag just like our good old puzzles !

Here are my GIFs (sorry they are tiny):
![](_attachments/Pasted%20image%2020240406152826.png)
![](_attachments/Pasted%20image%2020240406152803.png)
![](_attachments/Pasted%20image%2020240406152848.png)

And the flag i rebuilt:
![](_attachments/Pasted%20image%2020240406155234.png)
(Forgot about the FCSC part since i knew it already)
FCSC{L1ghtD3sign3rCr-gg!}
