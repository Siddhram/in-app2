# Firebase Setup for Expo

This guide will help you set up Firebase properly with your Expo app.

## The Problem

The error `this._firestore.native.documentSet is not a function` occurs because:

1. You're using Expo Go (which doesn't support native Firebase modules)
2. Firebase needs to be configured differently for Expo

## Solution Options

### Option 1: Use Expo Development Build (Recommended)

1. **Install EAS CLI**:

   ```bash
   npm install -g @expo/eas-cli
   ```

2. **Login to Expo**:

   ```bash
   eas login
   ```

3. **Configure EAS**:

   ```bash
   eas build:configure
   ```

4. **Create development build**:

   ```bash
   eas build --profile development --platform android
   # or for iOS
   eas build --profile development --platform ios
   ```

5. **Install the development build** on your device

### Option 2: Use Firebase Web SDK (Quick Fix)

Replace the Firebase service with web SDK:

1. **Install Firebase web packages**:

   ```bash
   npm install firebase
   ```

2. **Create Firebase config** (`firebaseConfig.js`):

   ```javascript
   import { initializeApp } from "firebase/app";
   import { getFirestore } from "firebase/firestore";

   const firebaseConfig = {
     apiKey: "your-api-key",
     authDomain: "your-project.firebaseapp.com",
     projectId: "your-project-id",
     storageBucket: "your-project.appspot.com",
     messagingSenderId: "123456789",
     appId: "your-app-id",
   };

   const app = initializeApp(firebaseConfig);
   export const db = getFirestore(app);
   ```

3. **Update firebaseService.ts** to use web SDK:

   ```typescript
   import { db } from "./firebaseConfig";
   import {
     doc,
     setDoc,
     addDoc,
     collection,
     serverTimestamp,
   } from "firebase/firestore";

   class FirebaseService {
     private getCurrentUserId(): string {
       return "demo-user-123";
     }

     async saveUserProfile(userData: any): Promise<void> {
       const userId = this.getCurrentUserId();
       const userRef = doc(db, "users", userId);

       await setDoc(userRef, {
         ...userData,
         profileCreatedAt: serverTimestamp(),
         lastUpdatedAt: serverTimestamp(),
       });
     }

     async savePersonalityTrait(traitData: any): Promise<string> {
       const userId = this.getCurrentUserId();
       const traitRef = collection(db, "users", userId, "PersonalityTraits");

       const docRef = await addDoc(traitRef, {
         ...traitData,
         timestamp: serverTimestamp(),
       });

       return docRef.id;
     }

     // Add other methods similarly...
   }

   export default new FirebaseService();
   ```

### Option 3: Use Mock Service (Current Setup)

The current setup uses a mock service that logs data to console. This is perfect for:

- Testing the UI
- Development without Firebase setup
- Learning the data structure

## Current Mock Service Features

The mock service currently:

- ✅ Logs all data to console
- ✅ Simulates async operations
- ✅ Shows success/error messages
- ✅ Maintains the same API as real Firebase

## Testing the Current Setup

1. **Run the app**:

   ```bash
   npm start
   ```

2. **Fill out the form** and click save buttons

3. **Check console logs** to see the data structure

4. **Data will be logged** like this:
   ```
   Mock Firebase: Saving user profile {email: "test@example.com", ...}
   Mock Firebase: User profile saved successfully
   ```

## Next Steps

1. **Choose your Firebase setup option** (recommend Option 1 or 2)
2. **Follow the setup guide** for your chosen option
3. **Replace the mock service** with real Firebase implementation
4. **Test with real Firebase** to ensure data is saved

## Firebase Project Setup

Regardless of which option you choose, you'll need:

1. **Firebase Project** at [console.firebase.google.com](https://console.firebase.google.com)
2. **Firestore Database** enabled
3. **Security Rules** configured
4. **Project configuration** (API keys, etc.)

## Security Rules for Development

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if true;
    }
  }
}
```

## Troubleshooting

### Common Issues:

- **Expo Go limitation**: Use development build instead
- **Network errors**: Check internet connection
- **Permission denied**: Update security rules
- **Configuration errors**: Verify Firebase config

### Debug Tips:

- Check console logs for detailed error messages
- Use Firebase Console to monitor operations
- Test with mock service first, then switch to real Firebase
