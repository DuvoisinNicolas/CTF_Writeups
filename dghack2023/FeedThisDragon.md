```
Voici le tout dernier jeu de type Clicker à la mode !
Il y a même un système de trophées mais l'un d'eux a l'air particulièrement difficile à obtenir...
Et l'obtenir semble débloquer une récompense secrète !
Mais est-ce "humainement" possible ? 
```

The game uses an API which we can see in network tab of the browser.

When we click on something, it changes our HTTP header, probably to pick the item we clicked on.

I've done this program:

```python
import requests
import json
import time


def send_request(headers):
    try:
        # Print information about the request
        #print(f"Sending GET request to with headers:")
       # print(json.dumps(headers, indent=2))

        response = requests.get("http://feedthisdragon2.chall.malicecyber.com/api/v1", headers=headers)
        response.raise_for_status()  # Raise an HTTPError for bad responses
        return response.json()
    except requests.exceptions.RequestException as e:
        return {"error": f"An error occurred: {e}"}
    # Process the JSON response
    #print(json.dumps(response_json, indent=2))

def send_api_request(headers, data=None):
    # Print information about the request
    #print("Sending POST request with headers:")
    # print(json.dumps(headers, indent=2))

    # Send the POST request
    response = requests.post("http://feedthisdragon2.chall.malicecyber.com/api/v1", headers=headers, data=data)
    response.raise_for_status()  # Raise an HTTPError for bad responses

    # Print the human-readable JSON response
    #print("Response:")
    response_json = response.json()
    # print(json.dumps(response_json, indent=2))

    return response_json


while(True):
    cookie = "2d5c44d2-94ad-41ee-b8d9-6c233bd1a0f6"
    HEADERS  = {
        'Cookie': "uuid=" + cookie,
        "Authorization": "mynotsosecrettoken",
        "Session": cookie
    }

    response_json = send_request(HEADERS)
    #print(json.dumps(response_json, indent=2))

    #print(response_json['items'])
    for item in response_json['items']:
        HEADERS = {
        "Content-Type":	"application/json",
        'Cookie': "uuid=" + cookie,
        "Authorization": "mynotsosecrettoken",
        "Session": cookie,
        "ItemUuid":	item['uuid'],
        "Update": "true",
        "ShopUuid" : ""
        }

        match item['type']:
            case "gem" | "coin" | "food" | "candy" | "burger" | "veggy" | "life" |"secret" | "lilboo" | "nyan":
                # Eat food or kill fox
                response = send_api_request(HEADERS)
                #print("clicking on the", item['type'])
            case "trap" | "fox" :
                pass
                # Do nothing
            case "midboo":
                response = send_api_request(HEADERS)
                response = send_api_request(HEADERS)
                #print("clicking on the", item['type'], "2 times")
                # Click twice
            case "bigboo":
                response = send_api_request(HEADERS)
                response = send_api_request(HEADERS)
                response = send_api_request(HEADERS)
                #print("clicking on the", item['type'], "3 times")
            case _:
                print(item["type"])

    #time.sleep(1)

    HEADERS = {
    "Content-Type":	"application/json",
    'Cookie': "uuid=" + cookie,
    "Authorization": "mynotsosecrettoken",
    "Session": cookie,
    "ItemUuid":	"",
    "Update": "true",
    "ShopUuid" : ""
    }
    for upgrade in response_json["upgrades"]:
        if response_json["bag"] >= 10000:
            break
        elif upgrade["cost"] <= response_json["coin"] and upgrade["name"] != "Hard" and upgrade["name"] != "Flee":
                HEADERS["ShopUuid"] = upgrade["uuid"]
                response = send_api_request(HEADERS)



```

`DGHACK{ThisDragonIsNowStuffed}`
