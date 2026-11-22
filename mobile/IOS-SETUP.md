# AIV X on the App Store: setup guide (no Mac needed)

The iPhone app is a native shell (Capacitor) that loads aiv-x.com. On top of the website it adds native Google and
Apple sign-in, push notifications, haptics, and an offline screen. GitHub Actions builds it on a cloud Mac and sends
it to TestFlight. Your website keeps working exactly as before, because all the app-only code checks `NATIVE` first.

Bundle ID: `com.aiverse.aivx`. If you change it, change it in `mobile/capacitor.config.json` and in
`mobile/ios/App/App.xcodeproj/project.pbxproj` (PRODUCT_BUNDLE_IDENTIFIER), and in the workflow's screenshot step.

---

## 1. Apple Developer: App ID and keys (developer.apple.com > Account)

1. **Certificates, IDs & Profiles > Identifiers > +** > App IDs > App.
   Description `AIV X`, Bundle ID (Explicit) `com.aiverse.aivx`.
   Under Capabilities, tick **Push Notifications** and **Sign In with Apple**. Continue > Register.
2. **Keys > +** > name it `AIV X Push`, tick **Apple Push Notifications service (APNs)** > Continue > Register.
   Download the `.p8` file (you can only download it once) and write down its **Key ID**.
3. **Membership details**: copy your **Team ID** (10 characters).

## 2. App Store Connect (appstoreconnect.apple.com)

1. **Apps > + > New App**: platform iOS, name `AIV X` (if taken, try `AIV X: Build Your AI`),
   language English (U.S.), bundle ID `com.aiverse.aivx`, SKU `aivx`.
2. **Users and Access > Integrations > App Store Connect API > Team Keys > +**.
   Name `GitHub builds`, access **Admin** (needed so the cloud Mac can create signing certificates).
   Download the `.p8`, and copy the **Key ID** and the **Issuer ID** shown above the list.

## 3. Firebase (console.firebase.google.com > project aiv-x-21aba)

1. **Project settings > General > Add app > iOS**. Bundle ID `com.aiverse.aivx`, nickname `AIV X iOS`.
   Download **GoogleService-Info.plist**. Skip the remaining steps in Firebase's wizard; they're already done.
   Open the file in Notepad and check it contains `CLIENT_ID` and `REVERSED_CLIENT_ID`. If not, check Google
   is turned on in Authentication > Sign-in method, then download the file again.
2. **Project settings > Cloud Messaging > Apple app configuration > APNs Authentication Key > Upload**:
   the push `.p8` from step 1.2, its Key ID, and your Team ID.
3. **Authentication > Sign-in method > Add new provider > Apple > Enable > Save.** (For the iPhone app you can
   leave the Services ID fields empty.)

## 4. GitHub secrets (github.com/domvq/AIV-X > Settings > Secrets and variables > Actions > New repository secret)

| Secret | What to paste |
|---|---|
| `APPLE_TEAM_ID` | your Team ID |
| `ASC_KEY_ID` | the App Store Connect API Key ID |
| `ASC_ISSUER_ID` | the Issuer ID |
| `ASC_KEY_P8` | open the API key `.p8` in Notepad and paste everything, including the BEGIN/END lines |
| `GOOGLE_SERVICE_INFO_PLIST` | open `GoogleService-Info.plist` in Notepad and paste everything |

Never put these files in the repo. `mobile/.gitignore` already blocks them.

## 5. Build and install on your iPhone

1. GitHub > **Actions > iOS app > Run workflow** (pick the branch, leave "Upload to TestFlight" ticked).
   A run takes about 15-25 minutes.
2. In App Store Connect > your app > **TestFlight**, the build appears 10-30 minutes after the run finishes.
   The first time, answer the export-compliance question (the app already declares it uses no special encryption).
3. TestFlight > **Internal Testing > +** > create a group, add yourself. Internal testers don't need Apple review.
4. On your iPhone, install **TestFlight** from the App Store, open the invite email, and install AIV X.

Each new run gets a new build number automatically. Website changes show up in the app right away, with no new
build, because the app loads aiv-x.com. You only need a new build when something in `mobile/` changes.

## 6. Sending notifications to everyone who has the app

Every iPhone that turns notifications on joins the topic **`all`** (and `ios`).

Firebase console > **Messaging > New campaign > Notifications**: write the title and text, then
Target > **Topic** > `all`. Schedule it or send it now.

Optional "Additional options > Custom data" controls what opens when someone taps:
- `app` = `study`, `finance`, `fitness`, `photo`, `career`, `music` or `aide` opens that AI Verse app
- `url` = any `https://aiv-x.com/...` link

Signed-in users' devices are also saved in Firestore at `users/{uid}/devices/{id}` (`token`, `platform`), so later
`aivx-worker.js` can send one person a notification (for example "Your agent report is ready") through the FCM
HTTP v1 API with a Firebase service account.

The app asks about notifications 4 seconds after someone signs in, with an in-app question first. Only if they
tap "Turn on" does iOS show its own permission prompt. It only asks once.

## 7. Before you press "Submit for Review"

- [ ] **AI providers:** check that `AI_PROVIDERS` in `index.html` lists every AI service `aivx-worker.js` sends
      messages to (now: "Cloudflare Workers AI and Groq"). Add those services, plus push notifications, to
      `privacy.html`. Apple rejects apps that send data to AI without naming who gets it (rule 5.1.2(i)).
- [ ] **App Privacy** (App Store Connect): declare contact info, user content (chats, photos, audio),
      identifiers (user ID, device token), and that user content is used to improve the product, linked to the user.
- [ ] **Reviewer account:** App Review Information > sign-in required > give an email and password account that works.
- [ ] **Where it's sold:** start with the **United States only** (Pricing and Availability). "Get AIV Pro" opens
      aiverse.it.com in Safari, and only the US App Store allows linking out to buy without Apple In-App Purchase.
      Other countries need In-App Purchase first.
- [ ] **Screenshots:** 6.9-inch iPhone (1320 x 2868). Run the workflow with "Also take App Store screenshots" for
      the first one, or take them on your iPhone and resize them. Show real features: chat, Build, the AI Verse apps.
- [ ] **Age rating:** answer the questionnaire honestly (open-ended AI chat). Your Terms say 18+.
- [ ] **Listing text:** describe what AIV X does. Leave out partnership claims (OpenAI, xAI) unless there's a
      signed agreement, and don't name other apps (ChatGPT, Grok) in the keywords.
- [ ] Support URL (`https://aiverse.it.com/#contact` or similar) and Privacy Policy URL (`https://aiv-x.com/privacy.html`).

## What's where

| File | What it does |
|---|---|
| `mobile/capacitor.config.json` | app ID, name, loads aiv-x.com, plugin settings |
| `mobile/ios/App/App/AppDelegate.swift` | hands push tokens and sign-in links to the plugins |
| `mobile/ios/App/App/App.entitlements` | Push Notifications + Sign in with Apple |
| `mobile/ios/App/App/Info.plist` | permission texts, portrait only, background push, Google sign-in link |
| `mobile/www/offline.html` | shown when the phone has no connection |
| `.github/workflows/ios.yml` | cloud Mac build > TestFlight, and Simulator screenshots |
| `index.html` (search `NATIVE`) | native sign-in, AI consent screen, notifications, haptics |
