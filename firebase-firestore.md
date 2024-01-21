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

## Add and Update Data

### setDoc

```javascript
import { doc, setDoc } from "firebase/firestore";

// if not exists then will be created new one
const cityRef = doc(db, "cities", "slmn");
const city = await setDoc(cityRef, {
  name: "Sleman",
  state: "Daerah Istimewa Yogyakarta",
  country: "Indonesia",
});

console.log(city); // undefined
```

If the document does not exist, it will be created. If the document does exist, its contents will be overwritten with the newly provided data, unless you specify that the data should be merged into the existing document, as follows:

```javascript
await setDoc(
  cityRef,
  {
    capital: false,
  },
  { merge: true }
);
```

### addDoc

addDoc is similar to setDoc, the difference is, addDoc does not require an ID in the creation process because it has been created automatically by cloud firestore and addDoc will return a DocumentReference while setDoc does not

```javascript
const city = await addDoc(collection(db, "cities"), {
  name: "new city",
  state: "new state",
  country: "new country",
});

console.log(city); // DocumentReference;
```

### updateDoc

```javascript
import {
  doc,
  updateDoc,
  serverTimestamp,
  arrayUnion,
  increment,
} from "firebase/firestore";

const cityRef = doc(db, "cities", "slmn");
const city = await updateDoc(cityRef, {
  state: "Daerah Istimewa Yogyakarta",
  updatedAt: serverTimestamp(), // serverTimestamp() which tracks when the server receives the update

  // to update nested object use dot notation to prevent overwrite the entire map field
  "geo.latitude": "-7.7470",
  "geo.longitude": "110.3756",

  // If your document contains an array field, you can use arrayUnion() and arrayRemove() to add and remove elements
  regions: arrayUnion("Seyegan"), // add to element if not exists
  population: increment(1000),
});

console.log(city); // undefined
```

## Delete Data

### deleteDoc

When you delete a document, Cloud Firestore does not automatically delete the documents within its subcollections. You can still access the subcollection documents by reference. For example, you can access the document at path `/mycoll/mydoc/mysubcoll/mysubdoc` even if you delete the ancestor document at `/mycoll/mydoc`.

```javascript
import { doc, deleteDoc } from "firebase/firestore";

const cityRef = doc(db, "cities", "deleted");
const deleted = await deleteDoc(cityRef);
console.log(deleted);
```

### deleteField

```javascript
import { updateDoc, deleteField } from "firebase/firestore";

await updateDoc(cityRef, {
  capital: deleteField(),
});
```

## Transactions and Batched Writes

### Transactions

Transcation function might run more than once if a concurrent edit affects a document that the transaction reads. The transaction read a document that was modified outside of the transaction. In this case, the transaction automatically runs again. The transaction is retried a finite number of times.

```javascript
import { doc, runTransaction, udpateDoc } from "firebase/firestore";

const likes = async () => {
  const postRef = doc(db, "posts", "post1");

  try {
    const newLikes = await runTransaction(db, async (transaction) => {
      // Read operations must come before write operations, otherwise the transaction will fail
      const post = await transaction.get(postRef);
      if (!post.exists()) {
        throw new Error("post doesnt exists");
      }

      const likeCount = post.data().likes + 1;
      // Simulates documents that are changed outside of the transaction (race condition)
      if (Math.random() < 0.5) {
        console.log("external update");
        await updateDoc(postRef, { likes: likeCount });
      }

      transaction.update(postRef, { likes: likeCount });
      return likeCount;
    });

    console.log(newLikes);
  } catch (error) {
    console.log(error);
  }
};
```

### Batched Writes

Different from transactions but having the same function, Batched Writes are used only to write data, not to read data

```javascript
import { writeBatch, doc } from "firebase/firestore";

const batch = writeBatch(db);

const postRef = doc(db, "posts/post1");
batch.set(postRef, { title: "Cool Post Title" }, { merge: true });

const userRef = doc(db, "users/user1");
batch.update(userRef, { money: 5000 });

const cityRef = doc(db, "cities/oIy8jdLavthyGj0qPpk7");
batch.delete(cityRef);

await batch.commit();
```

## Read Data

### Get single document

```javascript
import { doc, getDoc } from "firebase/firestore";

const docRef = doc(db, "cities", "SF");
const docSnap = await getDoc(docRef); // DocumentSnapshot
```

### Get all documents

```javascript
import { getDocs, collection } from "firebase/firestore";

const cities = await getDocs(collection(db, "cities")); // QuerySnapshot
cities.forEach((city) => {
  console.log(city.data());
});
```

### Get subcollection

```javascript
import { getDocs, collection } from "firebase/firestore";

// reviews are a subcollection of users
const userReviews = await getDocs(collection(db, "users/alovelace/reviews")); // setup the correct path
userReviews.forEach((review) => {
  console.log(review.data());
});
```

### Query

#### Query operators

```javascript
const q = query(reference, whereConditions));
```

The `where()` method takes three parameters: a field to filter on, a comparison operator, and a value. Cloud Firestore supports the following comparison operators:

- `<` less than
- `<=` less than or equal to
- `==` equal to
- `>` greater than
- `>=` greater than or equal to
- `!=` [not equal to](https://firebase.google.com/docs/firestore/query-data/queries#not_equal)
- [array-contains](https://firebase.google.com/docs/firestore/query-data/queries#array_membership)
- [array-contains-any](https://firebase.google.com/docs/firestore/query-data/queries#in_and_array-contains-any)
- [in](https://firebase.google.com/docs/firestore/query-data/queries#in_and_array-contains-any)
- [not-in](https://firebase.google.com/docs/firestore/query-data/queries#in_and_array-contains-any)

#### Using query

```javascript
import { collection, query, where, getDocs } from "firebase/firestore";

const citiesRef = collection(db, "cities");
const q = query(citiesRef, where("capital", "==", true));
const cities = await getDocs(q);
cities.forEach((city) => {
  console.log(city.data());
});
```

#### Compound

```javascript
import { query, where } from "firebase/firestore";

// and
const q1 = query(
  citiesRef,
  where("state", "==", "CO"),
  where("name", "==", "Denver")
);
```

```javascript
import { query, or, where } from "firebase/firestore";

// or
const q = query(
  citiesRef,
  or(where("capital", "==", true), where("population", ">=", 1000000))
);
```

```javascript
import { query, collection, and, where, or } from "firebase/firestore";

// and with or
const q = query(
  collection(db, "cities"),
  and(
    where("state", "==", "CA"),
    or(where("capital", "==", true), where("population", ">=", 1000000))
  )
);
```

### Order and Limit

```javascript
import {
  collection,
  query,
  where,
  orderBy,
  limit,
  getDocs,
} from "firebase/firestore";

const citiesRef = collection(db, "cities");
// ascending will be the default value of orderBy if not specified
const q = query(
  citiesRef,
  where("population", ">", 100000),
  orderBy("population", "desc"),
  limit(2)
);
const cities = await getDocs(q);
```

### Aggregation

```javascript
import {
  collection,
  getAggregateFromServer,
  count,
  sum,
  average,
} from "firebase/firestore";

const coll = collection(db, "cities");
// AggregateQuerySnapshot
const snapshot = await getAggregateFromServer(coll, {
  countOfDocs: count(),
  totalPopulation: sum("population"),
  averagePopulation: average("population"),
});
```

### Pagination

Use the `startAt()` or `startAfter()` methods to define the start point for a query. The `startAt()` method includes the start point, while the `startAfter()` method excludes it. For example, if you use `startAt(A)` in a query, it returns the entire alphabet. If you use `startAfter(A)` instead, it returns `B-Z`.

```javascript
import { collection, orderBy, query startAfter, limit, getDocs } from "firebase/firestore"

const paginate = async (cursor) => {
  const citiesRef = collection(db, "cities");
  let q = query(citiesRef, orderBy("population"), limit(2));
  if (typeof cursor !== "undefined") {
    console.log("page changed");
    q = query(citiesRef, orderBy("population"), startAfter(cursor), limit(2));
  }

  const cities = await getDocs(q);
  cities.forEach((city) => {
    console.log(city.data());
  });

  // DocumentSnapshot can be query cursor
  const nextCursor = cities.docs[cities.docs.length - 1];
  console.log(nextCursor);
  return nextCursor;
};

const runPagination = async () => {
  try {
    const cursor = await paginate();
    paginate(cursor);
  } catch (error) {
    console.log(error);
  }
};
```

### Real time updates

To listen for changes to the database, use the `onSnapshot()` method, this method will be called automatically if there are changes to the associated document. This method will return the `unsub` method which is used to stop listening for database changes. You can also include an error callback in the 3rd parameter

#### Listen to single document

```javascript
import { onSnapshot, doc } from "firebase/firestore";

const unsub = onSnapshot(
  doc(db, "users/user1"),
  (doc) => {
    // Retrieved documents have a metadata.hasPendingWrites property that indicates whether the document has local changes that haven't been written to the backend yet
    // This is because of an important feature called "latency compensation."
    // When you perform a write, your listeners will be notified with the new data before the data is sent to the backend.
    const source = doc.metadata.hasPendingWrites ? "local" : "server";
    console.log(source, " data: ", doc.data());
  },
  (error) => {
    // error callback
    console.log(error);
  }
);

// call the unsub method to stop listening for changes
unsub();
```

#### Listen to multiple documents

```javascript
import { query, collection, where, onSnapshot } from "firebase/firestore";

const q = query(collection(db, "cities"), where("state", "==", "CA"));
onSnapshot(q, (docs) => {
  docs.docChanges().forEach((change) => {
    if (change.type === "added") {
      console.log("New city: ", change.doc.data());
    }
    if (change.type === "modified") {
      console.log("Modified city: ", change.doc.data());
    }
    if (change.type === "removed") {
      console.log("Removed city: ", change.doc.data());
    }
  });

  const cities = [];
  docs.forEach((doc) => {
    cities.push(doc.data().name);
  });

  console.log("Current cities in CA: ", cities.join(", "));
});
```

## Security Rules

Security rules will consist of defining what specific documents you secure and what logic you use to secure them. Security rules can be bypassed via server side apps such as Go, NodeJS, Python etc. This can be prevented by implementing [IAM](https://cloud.google.com/firestore/docs/security/iam)

### Nested rules

Even though it can be nested, the rules applied to the parent will not affect the child because it has a different path, unless you use wildcards `=**`

```javascript
service cloud.firestore {
  match /database/{database}/documents {
    // path = users/123
    match /users/{userID} {
      allow read, write

      // path = users/123/reviews/456
      match  /reviews/{reviewID} {
        allow read
      }

      // path = users/123/private-data/456
      match /private-data/{privateDoc} {
        allow read
      }
    }
  }
}
```

### Wildcard

#### Single element

```javascript
match /users/{userID} {
  // userID = The ID of the user doc
  match /reviews/{reviewID} {
    // reviewID = The ID of the review doc
    // you can use userID in child
  }
}
```

#### Catch all

Please be careful when using this wildcard because it will affect access on other paths

```javascript
match /{document=**} {
  // document can match anything path
  // example
  // users/123
  // users/123/reviews/456
}
```

### Request

This object contains information from incoming requests

#### Auth

The auth object contains information about the logged in user

```javascript
// token is JWT claim object
match /myCollection/{docID} {
  allow read: if request.auth != null &&
    request.auth.token.email_verified
}
```

#### Resource

Resource objects refer to objects that represent documents to be written or updated in the database. The actual content is in `request.resource.data`, this object contains data that will be sent or updated by the user through operations such as creating a new document or updating an existing document.

```javascript
match /reviews/{reviewID} {
  allow create: if request.resource.data.score is number &&
    request.resource.data.score >= 1 &&
    request.resource.data.score <= 5 &&
    request.resource.data.review is string &&
    request.resource.data.review.size() > 20 &&
    request.resource.data.review.size() <=1000;
}
```

### Resource

Different from the `request.resource` object, the resource object refers to an object that represents a document that already exists in the database

```javascript
match /reviews/{reviewID} {
  allow update: if request.resource.data.score is number &&
    request.resource.data.score >= 1 &&
    request.resource.data.score <= 5 &&
    request.resource.data.review is string &&
    request.resource.data.review.size() > 20 &&
    request.resource.data.review.size() <=1000 &&
    // only the author can edit his own review
    resource.data.reviewerID == request.auth.uid &&
    // cant change the score
    request.resource.data.score == resource.data.score
}
```

### Arbritrary document

Arbitrary document refers to any document that may exist in the database. Sometimes you need to evaluate and apply rules to these documents in general, without regard to the specific structure or content of the document. For example, you might want to check whether the document exists or whether the user has certain access permissions to the document. This often involves using built-in functions such as `exists` or `get` to check if a document exists or get the document

```javascript
match /restaurants/{restaurantID} {
  // The dollar sign indicates that you will use a variable
  allow update: if get(/database/$(database)/documents/restaurants/$(restaurantID)/private_data/private).data.roles[request.auth.uid] == "editor";
}
```

### Function

Functions are used to help organize and abstract the logic of your security rules. This function allows you to understand the logic of security rules in a more structured form, making security rules easier to read, understand, and maintain.

```javascript
match /reviews/{reviewID} {
  function resourceScoreIsValid() {
    return request.resource.data.score is number &&
      request.resource.data.score >= 1 &&
      request.resource.data.score <= 5;
  }

  function resourceReviewIsValid() {
    return request.resource.data.review is string &&
      request.resource.data.review.size() > 20 &&
      request.resource.data.review.size() <=1000;
  }

  function resourceStateFieldIsValid() {
    return request.resource.data.state in ["draft", "published"]
  }

  function resourceIsValidReview() {
    return resourceScoreIsValid() &&
      resourceReviewIsValid() &&
      resourceStateFieldIsValid();
  }

  allow create: if resourceIsValidReview();

  allow update: if resourceIsValidReview() &&
    resource.data.reviewerID == request.auth.uid &&
    request.resource.data.score == resource.data.score;
}
```
