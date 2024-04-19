Since the challenge is named castafiore, i think that there are frequencies that are too high for me to hear.

So i asked chatGPT to make this in a more hearable file.

```python
from pydub import AudioSegment
import numpy as np

def read_wav(filename):
    """Reads a WAV file and returns the audio data and sample rate."""
    sound = AudioSegment.from_file(filename)
    return sound.get_array_of_samples(), sound.frame_rate

def pitch_shift(audio_data, sample_rate, shift_factor):
    """Shifts the pitch of audio data by a specified factor."""
    sound = AudioSegment(
        audio_data.tobytes(),
        frame_rate=sample_rate,
        sample_width=audio_data.itemsize,
        channels=1
    )
    shifted_sound = sound._spawn(
        (np.array(sound.get_array_of_samples()).astype(np.float32) * shift_factor).astype(np.int16)
    )
    return shifted_sound

if __name__ == "__main__":
    # Provide the path to your WAV file
    input_file = "silent.wav"

    # Read the WAV file
    audio_data, sample_rate = read_wav(input_file)

    # Define the pitch shift factor (e.g., 0.5 to shift one octave down)
    pitch_shift_factor = 0.5

    # Perform pitch shifting
    shifted_audio = pitch_shift(audio_data, sample_rate, pitch_shift_factor)

    # Save the shifted audio to a new WAV file
    output_file = "noisy.wav"
    shifted_audio.export(output_file, format="wav")

    print(f"Ultrasound audio saved to {output_file}")

```

Once this is done, on the file "noisy.wav", i ran Audacity and messed around with parameters.

I changed the pitch, and i started hearing something looking like "FCSC", so i messed a bit with this parameter, until i found a good value: 16.25 semitones.

The preview was "clear" but when applying changes to the file, the sound was changing.
So i increased the duration of the preview in parameters, in order to hear the whole flag.

And i flagged ! :)

`FCSC{2035351597102220198194}`