# Description

This is the specification for the HackBout project. Only the major features will be implemented in a prototypical manner during the hackathon due to time constraints. This means that accounts may be left out, financial data may be left unencrypted, and less flexible technologies such as iOS may be replaced with more prototyping friendly ones such as the web.

At the end of the hackathon, before the judging starts, create a presentation based off of this document. Thoroughly explain the entire idea, even the parts which weren’t implemented (those will be clearly marked as having been left out).

The description below describes the setup a real world application requires. For the hackathon, the following modifications will be made.
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
- Since we are not storing real data, simply persisting objects between program executions will suffice. Instead of using a real DBMS, MongoDB will be used as an object dump. There is no need to maintain uniqueness in trivial matters such as bus stops. It will neither break the application nor negatively affect the demo should two bus stops happen to use the same name. A much simpler schema will be used, which will look like [this example](db.json5) (each top-level key will warrant its own collection).