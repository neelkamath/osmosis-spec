# Caveats

The rest of the specification documents in this repository explain what the production application will behave like. However, only the major features will be implemented in the hackathon. This document explains the deviations made from the original idea for the sake of the hackathon's demo. Accounts may be left out, financial data may be left unencrypted, and less flexible technologies such as iOS may be replaced with more prototyping friendly ones such as the web.

At the end of the hackathon, before the judging starts, a presentation based off of these specification documents will be created. Thoroughly explain the entire idea, even the parts which weren’t implemented (those will be clearly marked as having been left out).

- Any account-related activity, such as logging in or deleting an account, will be left out.
- Unless there are very good reasons for creating mobile apps, web apps should be made instead so as to prevent running into the time-consuming issues one all too often faces when creating mobile apps.
- Here are the suggested technologies to use. These are only recommendations. If the hacker has good reason to use others, they may do so sans discussion.
    - Common
        - VCS: GitHub
    - Frontend
        - Programming language: TSX
        - Templating language: HTML
        - Styling language: CSS
        - Library: React
        - Bundler: Parcel
        - Hosting: Netlify
        - UI components: rmwc
        - QR code reader: instascan
        - QR code generator: qart.js.
    - Backend
        - Language: Kotlin
        - Framework: ktor
        - Deployment: Docker
        - DBMS: MongoDB
        - Server hosting: Heroku
        - DBMS hosting: mLab (through a Heroku addon)
- The recommended technologies to use on the backend are Kotlin, Heroku
- Financial transactions will be faked using plain arithmetic. This will be stated during the demo.
- The admin’s panel will implement the stated charts in order of priority. Assigning priorities will be up to the hacker, and will depend on how long it’ll take to learn and implement each chart. It is acceptable if a few of the admin panel’s features are missing in the case of time running out.

## Database

Since we are not storing real data, simply persisting objects between program executions will suffice. Instead of using a real DBMS, MongoDB will be used as an object dump. There is no need to maintain uniqueness in trivial matters such as bus stops. It will neither break the application nor negatively affect the demo should two bus stops happen to use the same name.

A much simpler schema will be used. The amount to be added as a fine will be hardcoded to a value such as 20. Each of the following subheadings will document the structure of MongoDB collections and MongoDB documents.

### Balance

The `balance` collection will contain a single document holding the user's balance. It will look like the following.

```json
{
    "balance": 350
}
```

### Bus Routes

Every document in the `bus_routes` collection stores a bus route. Here are two examples.

```json5
{
    // Route name
    "route": "blue",
    // The bus will travel through the following stops in a cyclic order
    "stops": [
        {
            "name": "Silk Board",
            "longitude": 31.67,
            "latitude": 78.54
        },
        {
            "name": "Jayadeva",
            "longitude": 34.72,
            "latitude": 79.99
        },
        {
            "name": "Banashankari",
            "longitude": 37.22,
            "latitude": 80.02
        }
    ],
    // Cost matrix
    "costs": [
        [
            0,
            20,
            25
        ],
        [
            50,
            0,
            20
        ],
        [
            30,
            40,
          0
        ]
    ]
}
```

```json
{
    "route": "green",
    "stops": [
        {
            "name": "Marenahalli",
            "longitude": 50.22,
            "latitude": 80.02
        },
        {
            "name": "BTM Layout",
            "longitude": 51.24,
            "latitude": 81.07
        },
        {
            "name": "Koramangala",
            "longitude": 52.32,
            "latitude": 83.02
        }
    ],
    "costs": [
        [
            0,
            20,
            30
        ],
        [
            40,
            0,
            15
        ],
        [
            10,
            15,
            0
        ]
    ]
}
```

### Buses

Every document in the `buses` collection stores a bus's metadata. Here are two examples.

```json5
{
    "bus": "KAI5682",
    "route": "blue",
    // Current longitude
    "longitude": 31.67,
    // Current latitude
    "latitude": 78.54
}
```

```json
{
    "bus": "KAI9723",
    "route": "green",
    "longitude": 51.24,
    "latitude": 81.07
}
```

### Tickets

Every document in the `tickets` collection stores a single ticket.

Here's an example of a completed transaction from a cash payer.
```json5
{
    // The bus the passenger is on
    "bus": "KAI5682",
    // Where and when the passenger got on
    "pickup": {
        "date": "2020-01-17",
        "time": "12:16",
        "location": "Banashankari"
    },
    // Where and when the passenger got off, or will get off
    "drop": {
        // This will be null if the passenger receives a fine.
        "date": "2020-01-17",
        // This will be null if the passenger receives a fine.
        "time": "13:20",
        // This will be null if the passenger receives a fine.
        "location": "Silk Board"
    },
    "transaction": {
        // true if the passenger paid in cash, false if paid via the app
        "cash": true,
        // Number of tickets the passenger requested
        "tickets": 2,
        // Whether the passenger is currently traveling
        "ongoing": false,
        /*
        This is whether the fee includes a fine the passenger had to pay. This will be null if the transaction hasn't
        ended yet.
        */
        "fined": false,
        // This is how much the passenger paid. It will be null if the transaction is ongoing.
        "fee": 60
    }
}
```

Here's an example of a completed transaction paid via the app.
```json
{
    "bus": "KAI5682",
    "pickup": {
        "date": "2020-01-14",
        "time": "13:36",
        "location": "Jayadeva"
    },
    "drop": {
        "date": null,
        "time": null,
        "location": null
    },
    "transaction": {
        "mode": "app",
        "tickets": 3,
        "ongoing": true,
        "fined": null,
        "fee": null
    }
}
```

Here's an example of a completed transaction of someone who paid via the app.
```json
{
    "bus": "KAI5682",
    "pickup": {
        "date": "2020-02-25",
        "time": "23:45",
        "location": "Silk Board"
    },
    "drop": {
        "date": "2020-02-26",
        "time": "01:25",
        "location": "Banashankari"
    },
    "transaction": {
        "mode": "app",
        "tickets": 2,
        "ongoing": false,
        "fined": false,
        "fee": 50
    }
}
```

Here's an example of someone who was fined.
```json
{
    "bus": "KAI9723",
    "pickup": {
        "date": "2020-02-23",
        "time": "08:15",
        "location": "BTM Layout"
    },
    "drop": {
        "date": null,
        "time": null,
        "location": null
    },
    "transaction": {
        "mode": "app",
        "tickets": 1,
        "ongoing": false,
        "fined": true,
        "fee": 60
    }
}
```