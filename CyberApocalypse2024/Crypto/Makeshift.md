Simple code to reverse the algorithm used to encrypt:
```python
encrypted_flag = "!?}De!e3d_5n_nipaOw_3eTR3bt4{_THB"

new_flag = ''

for i in range(0, len(encrypted_flag), 3):
    new_flag += encrypted_flag[i+2]
    new_flag += encrypted_flag[i]
    new_flag += encrypted_flag[i+1]
    print(new_flag)
    
flag = new_flag[::-1]

print(flag)
```

Flag: HTB{4_b3tTeR_w3apOn_i5_n3edeD!?!}