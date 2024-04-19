First part is simply bypassing the 403 filter by using another URL to get a token:

`/api/v1/get_ticket -> /api/v1/./get_ticket`

Second part was exploiting this vuln: 
https://sploitus.com/exploit?id=EA5AF022-9630-5498-B523-B8505AD5BCA6

And we get the flag ! :)

