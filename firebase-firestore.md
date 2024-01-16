<h1 align="center">Firebase Firestore</h1>

## Initialize Firestore

```javascript
import { initializeApp } from "firebase/app";
import { getFirestore } from "firebase/firestore";

const firebaseConfig = {
  apiKey: "",
  authDomain: "",
  projectId: "",
  storageBucket: "",
  messagingSenderId: "",
  appId: "",
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

export default db;
```

## Data Model

### Document

In Cloud Firestore, the unit of storage is the document. Firestore has a maximum document size of 1 MiB. This includes the document's field names, values, and any nested maps or arrays.

```json
{
    "first": "Ada"
    "last": "Lovelace"
    "born": 1815
}
```

### Maps

Complex nested objects in a document are called maps. For example, you could structure the user's name from the example above with a map, like this:

```json
{
    "name": {
        "first" : "Ada"
        "last" : "Lovelace"
    },
    "born": 1815
}
```

### Collections

Documents live in collections, which are simply containers for documents.

```json
{
    "users": [
        "alovelace": {
            "first": "Ada"
            "last": "Lovelace"
            "born": 1815
        }
    ]
}
```

### Subcollection

A subcollection is a collection contained in a document. If you take a document where the document has a subcollection then you will not automatically get a subcollection of that document but must mention it explicitly

```json
{
  "users": [
    {
      "alovelace": {
        "name": "Ada Lovelace",
        "reviews": [
          "uniqueId": {
            "star": 5,
            "text": "lorem ipsum"
          }
        ]
      }
    }
  ]
}
```

### Reference

A reference is an object that represents a location in the database. References can be used to read, write, update, or delete data in that location. References can be represented as strings, for example "users/1234567890". This string represents the path to a document in the database.

#### Document Reference

```javascript
import { doc, getDoc } from "firebase/firestore";

const alovelaceRef = doc(db, "users", "alovelace"); // option 1
const alovelaceRef = doc(db, "users/alovelace"); // option 2
const alovelace = await getDoc(alovelaceRef); // will return document snapshot

const data = alovelace.data();
console.log(data);
```

#### Collection Reference

```javascript
import { collection, getDocs } from "firebase/firestore";

const usersCollectionRef = collection(db, "users");
const querySnapshot = await getDocs(usersCollectionRef);

// example of getting subcollection
const userReviewSubcollection = collection(db, "users/alovelace/reviews");
const reviews = await getDocs(userReviewSubcollection);

console.log(usersCollectionRef);
querySnapshot.forEach((user) => {
  // query snapshot contains QueryDocumentSnapshot
  // https://firebase.google.com/docs/reference/js/v8/firebase.firestore.QueryDocumentSnapshot
  console.log(user.data());
});
```

### Document Snapshot

A DocumentSnapshot contains data read from a document in your Firestore database. The data can be extracted with `.data()` or `.get(<field>)` to get a specific field. For a DocumentSnapshot that points to a non-existing document, any data access will return `undefined`. You can use the `exists` property to explicitly verify a document's existence.

### Query Snapshot

A QuerySnapshot contains zero or more `DocumentSnapshot` objects representing the results of a query. The documents can be accessed as an array via the `docs` property or enumerated using the forEach method. The number of documents can be determined via the `empty` and `size` properties.
