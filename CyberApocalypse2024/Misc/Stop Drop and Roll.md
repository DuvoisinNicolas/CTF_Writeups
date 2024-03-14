We get this prompt once connected to netcat.
```
===== THE FRAY: THE VIDEO GAME =====
Welcome!
This video game is very simple
You are a competitor in The Fray, running the GAUNTLET
I will give you one of three scenarios: GORGE, PHREAK or FIRE
You have to tell me if I need to STOP, DROP or ROLL
If I tell you there's a GORGE, you send back STOP
If I tell you there's a PHREAK, you send back DROP
If I tell you there's a FIRE, you send back ROLL
Sometimes, I will send back more than one! Like this: 
GORGE, FIRE, PHREAK
In this case, you need to send back STOP-ROLL-DROP!
Are you ready? (y/n) 
```

Seems to be simple scripting, i'll do it in python (with the help of chatgpt because i'm lazy (and efficient... :) )

```python
import socket
import time

def solve_challenge(ip, port):
    # Connect to the server
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.connect((ip, port))

    # Receive initial message
    initial_msg = s.recv(1024).decode()
    print("Server:", initial_msg)

    # Confirm readiness
    s.send(b'y\n')
    print("Sent: y")
    

    while True:
        # Receive scenario
        data = s.recv(1024).decode()
        print("Server:", data)

        # Parse scenario
        scenarios = data.split(', ')
        response = ''

        for scenario in scenarios:
            if 'GORGE' in scenario:
                response += 'STOP-'
            elif 'PHREAK' in scenario:
                response += 'DROP-'
            elif 'FIRE' in scenario:
                response += 'ROLL-'

        # Send response
        response = response.rstrip('-') + '\n'
        s.send(response.encode())
        print("Sent:", response.strip())

        # Introduce a delay
        time.sleep(0.1)
        
    # Close connection
    s.close()

if __name__ == "__main__":
    solve_challenge('94.237.62.195', 36096)
```

After some time, we get the Flag.
```
Server: Fantastic work! The flag is HTB{1_wiLl_sT0p_dR0p_4nD_r0Ll_mY_w4Y_oUt!}
```