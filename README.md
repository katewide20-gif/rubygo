# Ruby Website — Setup Guide

This site has real sign-up/login and stores customer data securely, using
two free services: **GitHub Pages** (hosting) and **Firebase** (accounts + database).

## Step 1: Create your free Firebase project
1. Go to https://console.firebase.google.com and sign in with a Google account.
2. Click **"Add project"**, name it (e.g. "ruby-lagos"), finish the setup steps.
3. Once inside the project, click the **Web icon (`</>`)** to add a web app.
4. Give it a nickname (e.g. "ruby-web"), skip "Firebase Hosting" checkbox.
5. Firebase will show you a `firebaseConfig` object. Copy it.

## Step 2: Paste your config into the site
1. Open `index.html`.
2. Find this block near the bottom (inside the `<script type="module">` tag):
   ```js
   const firebaseConfig = {
     apiKey: "YOUR_API_KEY",
     ...
   };
   ```
3. Replace it with the exact config Firebase gave you in Step 1.

## Step 3: Turn on Email/Password sign-in
1. In the Firebase console sidebar: **Build > Authentication > Get started**.
2. Click **Sign-in method** tab > **Email/Password** > enable it > Save.

## Step 4: Create the database
1. In the sidebar: **Build > Firestore Database > Create database**.
2. Choose **Start in production mode** > pick a location close to Nigeria (e.g. `eur3` or `nam5`) > Enable.
3. Go to the **Rules** tab and replace the contents with:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /users/{userId} {
         allow read, write: if request.auth != null && request.auth.uid == userId;
       }
       match /riders/{riderId} {
         allow read, write: if request.auth != null && request.auth.uid == riderId;
       }
       match /bookings/{bookingId} {
         allow create: if request.auth != null;
         allow read: if request.auth != null &&
           (request.auth.uid == resource.data.uid || request.auth.uid == resource.data.get('riderUid', null));
         allow list: if request.auth != null;
         allow update: if request.auth != null &&
           (request.auth.uid == resource.data.uid || request.auth.uid == resource.data.get('riderUid', null));

         match /messages/{messageId} {
           allow read: if request.auth != null;
           allow create: if request.auth != null && request.auth.uid == request.resource.data.senderUid;
         }
       }
     }
   }
   ```
4. Click **Publish**.

   This locks the database so each customer and rider can only touch bookings
   they're actually part of — nobody can read another person's data.

## Step 4.5: Add Google Maps (for live ride tracking)
This is optional — the site works fully without it, customers just won't
see a live map (they'll see a note instead).
1. Go to https://console.cloud.google.com and select the same project (it shares the name with your Firebase project, e.g. `ruby-54b40`).
2. In the search bar, type "Maps JavaScript API" and click **Enable**.
3. You'll be prompted to set up a **Billing account** — this requires a card,
   but Google gives $200/month free credit, which comfortably covers Ruby's
   early usage at no charge.
4. Go to **APIs & Services > Credentials > Create Credentials > API key**.
5. Copy the key, then open `index.html` and replace `YOUR_MAPS_API_KEY` in this line near the top:
   ```html
   <script src="https://maps.googleapis.com/maps/api/js?key=YOUR_MAPS_API_KEY" async defer></script>
   ```
6. For security, click on the key in the Credentials page and restrict it to your website's domain (e.g. `yourusername.github.io/*`).

**Important limitation:** live location only updates while the rider's
browser tab is open and location permission is granted for that session —
if they lock their phone or close the tab, sharing pauses until they reopen
it. This is a browser limitation, not a bug — true background tracking
(like Uber's native app) would need a real mobile app, not a website.

## Step 5: Put it on GitHub Pages (free hosting)
1. Create a free GitHub account at https://github.com if you don't have one.
2. Create a new repository (e.g. `ruby-website`) — make it **Public**.
3. Upload `index.html` to that repository (use "Add file > Upload files" on the GitHub website — no command line needed).
4. Go to the repo's **Settings > Pages**.
5. Under "Source," choose the `main` branch and `/ (root)` folder > Save.
6. GitHub gives you a live link like `https://yourusername.github.io/ruby-website/` within a minute or two.

## Note: one-click index prompt
The first time trip history loads, Firestore may show an error in the browser
console (press F12 to view it) asking to create an index. This is normal —
click the link it gives you, click "Create index" on the Firebase page that
opens, wait about a minute, then refresh your site. This only happens once.

## You're live
Share that link on WhatsApp, Facebook, Instagram — anyone who signs up gets
a secure account, and every booking they submit is saved safely to your
Firestore database (viewable in the Firebase console under **Firestore Database > Data**)
AND opens WhatsApp so you get it instantly too.

## Free tier limits (so you're not surprised later)
Firebase's free "Spark" plan comfortably covers a business your size starting out:
- 50,000 reads/day, 20,000 writes/day on Firestore
- Unlimited Authentication sign-ups
You won't need to pay anything until Ruby has serious volume — and by then you'll have revenue to cover it.
