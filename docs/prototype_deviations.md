# Prototype Deviations

The rest of the specification documents in this repository explain what the production application will behave like. However, only the major features will be implemented in the hackathon. This document explains the deviations made from the original idea for the sake of the hackathon's demo. Accounts may be left out, financial data may be left unencrypted, and less flexible technologies such as iOS may be replaced with more prototyping friendly ones such as the web.

At the end of the hackathon, before the judging starts, a presentation based off of these specification documents will be created. Thoroughly explain the entire idea, even the parts which weren’t implemented (those will be clearly marked as having been left out).

- Any account-related activity, such as logging in or deleting an account, will be left out.
- Unless there are very good reasons for creating mobile apps, web apps should be made instead so as to prevent running into the time-consuming issues one all too often faces when creating mobile apps.
- Here are the suggested technologies to use. These are only recommendations. If the hacker has good reason to use others, they may do so sans discussion.
    - Common
        - VCS: GitHub
    - Frontend
        - Programming language: [TSX](http://www.typescriptlang.org/docs/handbook/jsx.html)
        - Templating language: HTML
        - Styling language: CSS
        - Library: [React](https://reactjs.org/)
        - Bundler: [Parcel](https://parceljs.org/)
        - Hosting: [Netlify](https://www.netlify.com/)
        - UI components: [rmwc](https://github.com/jamesmfriedman/rmwc)
        - QR code reader: [instascan](https://github.com/schmich/instascan)
        - QR code generator: [qart.js](https://github.com/kciter/qart.js)
    - Backend
        - Language: Kotlin
        - Framework: [ktor](https://ktor.io/)
        - HTTP API tooling: OpenAPI
        - Deployment: Docker
        - DBMS: MongoDB
        - Server hosting: Heroku
        - DBMS hosting: mLab (through a Heroku addon)
- Bus movements will be faked server-side by updating bus locations automatically every 2 seconds to the coordinates of the next stop in their route.
- Users will be fined if they don't end a transaction within 10 seconds (instead of 24 hours).
- Passenger pickup and drop-off locations will be faked client-side by automatically randomly selecting a location on the current bus route.
- Financial transactions will be faked using plain arithmetic. This will be stated during the demo.
- Admins will not be able to CRUD bus routes, they'll be hardcoded instead.
- The admin panel will only display the following four analytics.
    - The total money transacted all time, and a line chart displaying the amount of money transacted over the past months.
    - A map of the real time locations of the buses, and how many passengers each one carries.
    - Bar charts for every bus route displaying the number of passengers that have traveled them over the months.
    - Every bus route will have its own heatmap. The heatmap will display the number of passengers who traveled the particular route. The rows and columns in the heatmap will be the times of the day and the last 7 days respectively.
- The DB will be seeded with hardcoded data for analytics (e.g., the number of passengers traveling on different bus routes in the previous months).

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
    "latitude": 78.54,
    // Number of seats (occupied and unoccupied) in the bus
    "seats": 50,
    "passengers": 20
}
```

```json
{
    "bus": "KAI9723",
    "route": "green",
    "longitude": 51.24,
    "latitude": 81.07,
    "seats": 70,
    "passengers": 75
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
        This is the amount the passenger was fined. This will be null if the transaction hasn't
        ended yet, 0 if the passenger wasn't fined, and a natural number if the user was fined.
        */
        "fine": 0,
        // This is how much the passenger paid. It will be null if the transaction is ongoing.
        "fare": 60
    }
}
```

Here's an example of an ongoing transaction paid via the app.
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
        "fine": null,
        "fare": null
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
        "fine": 0,
        "fare": 50
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
        "fine": 20,
        "fare": 60
    }
}
```