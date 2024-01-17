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
