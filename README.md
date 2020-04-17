# self-report backend

Firebase function for covid-self-report.

This firebase function will validate request using Google's reCaptcha v3 before inserting data into firestore, you must as such create a reCaptcha project (if you want to use it) and have firestore available in your Firebase project.

## Getting started
1. Create a firebase project if not already done
2. Clone this project
3. Update the `firebase.json` and `.firebaserc` files to reflect your configuration
4. Create a `.runtimeconfig.json` file based on `env.example.json` and update it with your values
5. Run `npm run env` from within the functions directory to populate your environment variables
6. Run `firebase serve` to test locally and `firebase deploy` to deploy it

### Configuration file
You have to manually create a `/functions/.runtimeconfig.json` file and set your backend config in it.
```js
{
  "host": {
    "domain": "", // Currently unused
    "region": "europe-west1" // Firebase region of hosting
  },
  "recaptcha": {
    "secret": "", // Recaptcha secret value (not public key)
    "verifyurl": "https://recaptcha.google.com/recaptcha/api/siteverify"
  },
  "db": {
    "report": "", // firestore collection in which legitimate reports go
    "suspicious": "", // firestore collection in which suspicious reports go
    "dev": "" // firestore collection used in development, currently not used
  },
  "export": {
    "token": "" // A token you can set to whatever value (we recommand using https://passwordsgenerator.net/ to generate a secure hash) this will be used to secure the export_json function
  }
}
```

## Firebase

### Firestore
After creating your firebase project, navigate to your database section and make sure to enable native firestore. This is required to be able to use the Firestore API from within functions.

### Rules
In order to use and secure your firestore, navigate to the rules tab of the firestore section, and write the following content:
```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow create: if request.auth.token.admin == true;
      allow read: if request.auth.token.admin == true;
      allow update: if request.auth.token.admin == true;
      allow delete: if request.auth.token.admin == true;
    }
  }
}
```
These rules make sure only admin can perform CRUD operations on your firestore, which is the case when a function is run. This ensures no one except your functions can access your firestore.

### Indexes
We encourage you to add indexes to your firestore collections, most importantly on the report collection. One index we put which seems to be effective is the following:
- Timestamp ASC
- Locator ASC
- diagnostic ASC


### Using firebase.json (optional)
Your `.firebase.json` file contains metadata about the project, including valuable informations if you want to host the frontend on Firebase. For it, you can use this simple configuration:
```json
{
    "hosting": {
        "public": "public",
        "ignore": [
        "firebase.json",
        "**/.*",
        "**/node_modules/**"
        ]
    }
}
```
Please note that the published website **must** be inside the /public directory. As such, you must first build the frontend and copy the dist/ content inside the public directory.
