Here's the code i used:
```python
import turtle

def draw_shape(segment_list, scale):
    turtle.speed(0)  # Set the drawing speed to the fastest
    x, y = 0, 0  # Initial position
    turtle.penup()
    turtle.goto(x * scale, y * scale)
    turtle.pendown()
    skip_next = False
    color_index = 0  # Initialize the color index
    for dx, dy in segment_list:
        if (dx, dy) == (0, 0):
            skip_next = True
        elif skip_next:
            x += dx
            y += dy * -1  # Multiply y-coordinate by -1 to revert on Y-axis
            skip_next = False
        else:
            turtle.penup()
            turtle.goto(x * scale, y * scale)
            x += dx
            y += dy * -1  # Multiply y-coordinate by -1 to revert on Y-axis
            turtle.pendown()
            turtle.goto(x * scale, y * scale)
            color_index += 1
    turtle.hideturtle()
    turtle.done()

def main():
    segments = [
        (0,2),(0,-2),(1,0),(-1,0),(0,1),(1,0),(0,0),(1,1),(0,-2),(1,0),
        (-1,0),(0,2),(1,0),(0,0),(2,-2),(-1,0),(0,1),(1,0),(0,1),(-1,0),
        (0,0),(2,0),(0,-2),(1,0),(-1,0),(0,2),(1,0),(0,0),(3,-2),(-1,0),
        (0,1),(-1,0),(1,0),(0,1),(1,0),(0,0),(4,-2),(-2,0),(0,0),(0,2),
        (2,0),(0,-2),(0,1),(-2,0),(0,0),(3,-1),(0,2),(0,0),(3,-2),(-1,0),
        (-1,1),(0,1),(2,0),(0,-1),(-2,0),(0,0),(3,0),(1,0),(0,-1),(-1,0),
        (0,2),(1,0),(0,-1),(0,0),(1,1),(1,0),(0,-2),(-1,0),(0,0),(0,1),
        (1,0),(0,0),(2,1),(0,-2),(-1,1),(2,0),(0,0),(1,-1),(1,0),(-1,2),
        (0,0),(0,-1),(1,0),(0,0),(1,-1),(1,0),(0,1),(-1,0),(0,1),(1,0),
        (0,0),(1,0),(1,0),(0,-1),(-1,0),(0,-1),(1,0),(0,0),(1,2),(0,-2),
        (1,0),(-1,0),(0,2),(1,0),(0,-1),(-1,0),(0,0),(2,1),(1,0),(-1,0),
        (0,-2),(1,0),(-1,2),(1,0),(0,-2),(0,0),(1,0),(0,1),(1,0),(0,-1),
        (0,2),(0,0),(2,-2),(1,0),(0,1),(1,0),(-1,0),(0,1),(-1,0)
    ]
    scale = 40  # Adjust the scale factor as needed
    draw_shape(segments, scale)

if __name__ == "__main__":
    main()

```

I reversed the coordinates to make it print in the right way, and used a scale factor to print it bigger.

![](_attachments/Pasted%20image%2020240406124655.png)


Flag is : FCSC{316834725604}