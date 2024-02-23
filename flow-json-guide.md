# Flow Config JSON Guide

## Main Data Structure

| Object | Type | Example | Description |
| --- | --- | --- | ----------- |
| flowName| String | Booking Ticket Flow | Name of the flow |
| ttl | String | 600s | Time to live of the chatbot session, the chatbot will close the session based on the time defined. Currently the chatbot engine only support seconds as the unit. In example you wan't to set the ttl to 5 minutes then the value will be `300s`  |
| unrecognizedMessage | String | I'm sorry, but I didn't quite understand your message. Could you please rephrase your request? | Default message when the user send an unexpected response when the session is opened  |
| finishedSessionMessage | String | Thank you for using our services | The message that will be send by the chatbot when the user reached the *endOfJourney*  |
| trigger | Array of String | ```["Hi", "Hello"]``` | Phrases that will trigger the chatbot if there is no session opened  |
| status | String | ```active``` | Status of the chatbot. Available statuses : `active`, `disabled`, `expired`  |
| steps | Array of Objects | `N/A` | List of the Steps / Journey of the chatbot. Refer to the [Steps Object](#steps-object) |

## Steps Object

| Object | Type | Example | Description |
| --- | --- | --- | ----------- |
| step_id | Integer | 1 | The identifier of the step |
| action | Object | [Action Example](#action-example) | Action field can be used to get a resource from external resources through HTTP connection  |
| responseType | String | `quick-reply` | Message type that will replied by the chatbot in each step |
| response | Object | [Response Example](#response-example) | Response details of the step |
| trigger | Array of String | ```["Hi", "Hello"]``` | Phrases that will trigger the chatbot if there is no session opened  |
| status | String | ```active``` | Status of the chatbot. Available statuses : `active`, `disabled`, `expired`  |
| steps | Array of Objects | `N/A` | List of the Steps / Journey of the chatbot. Refer to the [Steps Object](#steps-object) |

### Action

| Object | Type | Example | Description |
| --- | --- | --- | ----------- |
| url | String | `https://reqres.in/api/users?page=1` | Url of the endpoint target to fetch the result |
| method | String | `GET` | HTTP method to be used when calling the resources. Available data : `GET`, `PATCH`, `PUT`, `DELETE`, `POST`, `PUT` |
| headers | Object | `{"Content-Type": "application/json"}` | HTTP headers data that will be send when calling the endpoint |
| body | Object | `{"province": "Papua"}` | HTTP headers data that will be send when calling the endpoint |
| logic | Object | [Action Logic Example](#action-logic-example) | The logic to validate the data received from the endpoint |

#### Action Example

```json
{
  "url": "https://reqres.in/api/users?page=1",
  "method": "GET",
  "headers": {
    "Content-Type": "application/json"
  },
  "body": {},
  "logic": {}
}
```

#### Action Logic Example

To use the HTTP status code as validation you should use `[[response.status]]` and add the following standard JavaScript if operators.

**Example of using HTTP status code as the logic validator :**

```json
{
  "when": "[[response.status]] == 404",
  "then": {
    "responseType": "text",
    "response": {
      "text": {
        "body": "Sorry there is no available data for your request"
      }
    }
  }
}
```

**Example of using response body as the logic validator :**

Received body payload :

```json
{
  "data": {
    "id": 2,
    "email": "janet.weaver@reqres.in",
    "first_name": "Janet",
    "last_name": "Weaver",
    "avatar": "https://reqres.in/img/faces/2-image.jpg"
  },
  "support": {
    "url": "https://reqres.in/#support-heading",
    "text": "To keep ReqRes free, contributions towards server costs are appreciated!"
  }
}
```

Logic payload :

```json
{
  "when": "[[response.status]] == 404",
  "then": {
    "responseType": "text",
    "response": {
      "text": {
        "body": "Sorry there is no available data for your request"
      }
    }
  }
}
```

### Response

All the response should follow the `responseType` listed in each steps below are the list of available response type.

#### List of Available Response type

- [Quick Reply](#quick-reply) (quick-reply)
- [List](#list-reply) (list)
- [Location Request](#location-request) (location-request)
- [Call To Action URL](#call-to-action-url) (cta-url)
- [Carousel](#carousel) (template-carousel)
- [Templated Message](#templated-message) (template)
- [Text](#text-message) (text)

#### Quick Reply

##### Static Quick Reply

```json
{
  "body": {
    "text": "Apa yang bisa saya bantu hari ini?"
  },
  "footer": {},
  "buttons": [
    {
      "text": "Promo Terbaru",
      "payload": "promo",
      "nextStep": 2
    },
    {
      "text": "Bantuan CS",
      "payload": "bantuan-cs",
      "nextStep": 3
    }
  ]
}
```

##### Dynamic Quick Reply using Action Response

In this case the actions response data is an array of string.

###### Action Response Received

```json
{
  "page": 2,
  "per_page": 6,
  "total": 12,
  "total_pages": 2,
  "data": [
    {
      "id": 7,
      "email": "michael.lawson@reqres.in",
      "first_name": "Michael",
      "last_name": "Lawson",
      "avatar": "https://reqres.in/img/faces/7-image.jpg"
    },
    {
      "id": 8,
      "email": "lindsay.ferguson@reqres.in",
      "first_name": "Lindsay",
      "last_name": "Ferguson",
      "avatar": "https://reqres.in/img/faces/8-image.jpg"
    }
  ]
}
```

###### Quick Reply Response Example

```json
{
  "body": {
    "text": "Apa yang bisa saya bantu hari ini?"
  },
  "footer": {},
  "buttons": [
    {
      "text": "[[.data.first_name]]",
      "payload": "[[.data.email]]",
      "nextStep": 2
    }
  ]
}
```

#### List Reply

##### Static List Reply

```json
{
  "body": {
    "text": "Choose List Below"
  },
  "footer": {},
  "list": {
    "options": [
      {
        "text": "1",
        "payload": "1",
        "nextStep": 29
      },
      {
        "text": "2",
        "payload": "2",
        "nextStep": 29
      }
    ],
    "buttonText": "选择人数"
  }
}
```

##### Dynamic List Reply from Actions Response

The actions response used is referring to this data [Action Response](#action-response-received), this example below is going to loop for the data. Since the data in the action response is 2 data then it will create 2 selection options based on the `data` variable.

**Please be aware that the list only allowed to be 10 items maximum. If you put more than 10 items in the list it will caused an error and the chatbot will not be able to send the message to Meta*

###### List Reply Example

```json
{
  "body": {
    "text": "Choose List Below"
  },
  "footer": {},
  "list": {
    "options": [
      {
        "text": "[[data().first_name]]",
        "payload": "[[data().email]]",
        "nextStep": 29
      }
    ],
    "buttonText": "选择人数"
  }
}
```

#### Location Request

If you wish the customer to send a location you should set the `responseType` to `location-request`

##### Location Request Example

```json
{
 "body": {
  "text": "Please send us your location"
 },
 "nextStep": 4
}
```

#### Call to Action URL

Below is the rules when using CTA URL :

- Maximum CTA button is only 3
- CTA URL is commonly used only for ending a conversation

##### Response Example for CTA URL

```json
{
  "body": {
    "text": "Click URL below to see our latest Promotions!"
  },
  "action": {
    "name": "cta_url",
    "parameters": {
      "display_text": "View Promo",
      "url": "https://1engage.ai"
    }
  }
}
```

#### Templated Message

Basically to use a template as a response you just need to follow Meta message object in the payload but need to add the `nextStep` in each quick reply if you expect the user to be able to continue to the journey.

##### Response Example for Templated message

```json
{
  "template": {
    "language": "en",
    "templateName": "prod_movie_order_welcome",
    "components": [
      {
        "type": "header",
        "parameters": [
          {
            "type": "image",
            "image": {
              "link": "[[banner]]" //using a data returned from actions
            }
          }
        ]
      },
      {
        "type": "body",
        "parameters": [
          {
            "type": "text",
            "text": "[[movieList.0]]"
          },
          {
            "type": "text",
            "text": "[[movieList.1]]"
          },
          {
            "type": "text",
            "text": "[[movieList.2]]"
          },
          {
            "type": "text",
            "text": "[[movieList.3]]"
          },
          {
            "type": "text",
            "text": "[[movieList.4]]"
          }
        ]
      },
      {
        "nextStep": 2,
        "type": "button",
        "sub_type": "quick_reply",
        "index": "0",
        "parameters": [
          {
            "type": "payload",
            "payload": "showMoviesEn"
          }
        ]
      },
      {
        "nextStep": 15,
        "type": "button",
        "sub_type": "quick_reply",
        "index": "1",
        "parameters": [
          {
            "type": "payload",
            "payload": "showCinemaEn"
          }
        ]
      }
    ]
  }
}
```

#### Carousel

Before using the carousel in the flow make sure you already created a carousel template in meta dashboard.

##### Response Example for Carousel

```json
{
  "template": {
    "language": "en",
    "templateName": "prod_movie_carousel_p1",
    "components": [
      {
        "type": "carousel",
        "cards": [
          {
            "card_index": "0",
            "components": [
              {
                "type": "header",
                "parameters": [
                  {
                    "type": "image",
                    "image": {
                      "link": "https://static.wixstatic.com/media/38c00a_5a89549031c34427b8e07f83282a51d9~mv2.png/v1/fill/w_701,h_701,al_c/38c00a_5a89549031c34427b8e07f83282a51d9~mv2.png"
                    }
                  }
                ]
              },
              {
                "type": "body",
                "parameters": [
                  {
                    "type": "text",
                    "text": "Promo Agustus"
                  },
                ]
              },
              {
                "type": "button",
                "sub_type": "quick_reply",
                "index": "0",
                "nextStep": 12,
                "parameters": [
                  {
                    "type": "payload",
                    "payload": "Beli Sekarang"
                  }
                ]
              },
              {
                "type": "button",
                "sub_type": "quick_reply",
                "index": "1",
                "nextStep": 5,
                "parameters": [
                  {
                    "type": "payload",
                    "payload": "Beli Sekarang"
                  }
                ]
              }
            ]
          },
          {
            "card_index": "1",
            "components": [
              {
                "type": "header",
                "parameters": [
                  {
                    "type": "image",
                    "image": {
                      "link": "https://i.ibb.co/59nc745/e24c65b188324b22066dbea1a0b29e4c.png"
                    }
                  }
                ]
              },
              {
                "type": "body"
              },
              {
                "type": "button",
                "sub_type": "quick_reply",
                "index": "0",
                "nextStep": 13,
                "parameters": [
                  {
                    "type": "payload",
                    "payload": "Lihat Promo"
                  }
                ]
              },
              {
                "type": "button",
                "sub_type": "quick_reply",
                "index": "1",
                "nextStep": 3,
                "parameters": [
                  {
                    "type": "payload",
                    "payload": "Saya Tertarik"
                  }
                ]
              }
            ]
          }
        ]
      }
    ]
  }
}
```

#### Text Message

Text message will only response a text bubble, but you can still make the user to continue their flow journey by adding `nextStep` in the `response` object.

##### Response Example for Text Message

```json
{
  "text": {
    "preview_url": false,
    "body": "text-message-content",
    "nextStep": 2
  }
}
```