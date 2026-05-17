# Uncle's Pills Privacy Policy

**Last updated:** 2026-05-17
**Operator:** Martin Fu (Uncle's Pills)
**Contact:** unclespills-privacy@fufamily.com

This Privacy Policy explains how Uncle's Pills ("the app", "we", "us") handles
information when you use it on your iOS device. Uncle's Pills is a family
medication tracker. Unlike a fully local app, Uncle's Pills stores its data in
Google Firebase so that the members of one family can share a single
medication list across their phones. This document is specific about
what goes where.

## Accounts and authentication

Uncle's Pills uses Firebase Authentication to sign you in. You can choose one
of three providers:

- **Sign in with Apple.** Apple gives Firebase a stable user ID and,
  optionally, the email and name you choose to share (you can ask Apple
  to relay a hidden private email). Apple is your identity provider in
  this flow.
- **Sign in with Google.** Google gives Firebase your Google account's
  user ID, email, display name, and profile photo URL. Google is your
  identity provider in this flow.
- **Phone number.** You enter your phone number, Firebase texts you a
  6-digit code, and you type the code back in. Your phone number is
  stored on Firebase's servers as your account identifier. The SMS is
  delivered by Firebase / Google's telephony partners.

Whichever provider you pick, the resulting account is a single Firebase
user record identified by a Firebase UID. Uncle's Pills does not run its own
account server, set its own password, or see the password / face data
that you used to sign in with Apple or Google.

## What Uncle's Pills stores in the cloud

Once you sign in, Uncle's Pills stores the following data in **Google
Firestore** (a NoSQL database hosted by Google) and **Firebase Cloud
Storage** (Google Cloud Storage with Firebase rules in front), inside
the operator's Firebase project:

**Per-user document (`/users/{uid}`).** Your display name, profile
photo URL, language preference, the ID of the family you currently have
selected, and your Google AI Studio API key if you have entered one (see
"AI scan" below). This document is readable and writable only by you.

**Per-family document (`/families/{familyId}`).** A family represents a
shared medication list. The document holds the family name, the list of
member Firebase UIDs, the list of admin UIDs, the timestamp of when each
member joined, and the most recent active invite code (if any). Every
family member can read and update this document; only admins can delete
the family. There is no separate "household" concept — joining a family
means joining a shared dataset.

**Family snapshot (`/families/{familyId}/data/snapshot`).** All
medication-related data for one family lives in a single document:

- Each person tracked in the family (name, color, optional notes, sort
  order, avatar reference)
- Each medication (name in English and Chinese, dosage, schedule, time
  slots, "as needed" flag, expiry, prescription reference, notes)
- Each "pill box" (a logical grouping of pills, e.g. morning / evening)
- The dose log (which person took which medication at which time)
- Prescriptions (the original AI-scanned bag info, used for traceback)

This document is readable and writable by every member of the family.
Uncle's Pills does not encrypt fields inside the document — the Firestore
service holds them in plaintext, the way any Firestore document is
held, with TLS in transit and at-rest encryption managed by Google. The
security boundary is the family membership list: a user who is not in
the family cannot read the document.

**Avatar images (`/users/{uid}/avatar.jpg` in Firebase Storage).**
Square JPEGs at 512×512, capped at 2 MB. Any authenticated Uncle's Pills user
can read avatars (so other family members can see your face on the list
view); only you can write yours.

**Medicine-bag photos (`/families/{familyId}/photos/{id}` in Firebase
Storage).** When the AI scan flow runs (see below), the original photo
is uploaded so other family members can review the source image later.
Capped at 8 MB. Only members of the same family can read or write
photos under that family's path.

**Invite codes (`/invites/{code}`).** When you generate a join code,
Uncle's Pills writes a small lookup document with the family ID and an
expiration timestamp. The code is rotateable from inside the app. Any
authenticated user can read this lookup (that is how the join-by-code
flow works); only an admin of the target family can create one, and
the joiner deletes it after use.

## AI scan (Google Gemini)

Uncle's Pills includes an optional feature that uses Google's Gemini API to
read a photo of a medicine bag or prescription label and extract the
medication name, dosage, schedule, and quantity into structured form.

This feature is **bring-your-own-key**. You paste a Google AI Studio
API key into Settings → Account; that key is stored in your per-user
Firestore document and read by the app at request time. When you
press "Scan", the app does two things:

1. Uploads the original photo to Firebase Storage at
   `/families/{familyId}/photos/{id}`, so other members of the family
   can see the source later.
2. Sends the same image bytes, plus a prompt, directly from your
   device to
   `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent`
   using your own Google AI Studio key.

Google sees the photo and the prompt because Google operates the
Gemini API. We do not run a server in between, and we do not have an
arrangement with Google that gives us access to your AI usage. **We do
not send the photo or any of your data to Anthropic, OpenAI, or any
other AI vendor.** The only AI call the app makes is the Gemini call
described here.

If you do not configure a Gemini key, the AI scan feature is disabled
and the app never contacts Gemini at all.

## Device features the app uses

- **Camera.** When you tap "Scan medicine bag", iOS asks for camera
  permission so you can take the photo that becomes the AI input
  above. Camera frames are not streamed anywhere; the app receives a
  single still image after you tap the shutter.
- **Photo library.** As an alternative to the camera, you can pick an
  existing photo from your library. iOS asks for photo-library
  permission the first time you do this.
- **Network.** Uncle's Pills needs the internet to talk to Firebase
  (Firestore, Auth, Storage) and, optionally, to Google's Gemini API.
  All requests use TLS.
- **In-app sharing.** When you tap "Share invite link", Uncle's Pills hands
  the URL to the iOS share sheet using the Capacitor Share plugin so
  you can send it via Messages, Mail, etc. Uncle's Pills does not send the
  invite link anywhere on your behalf.

Uncle's Pills **does not** use:

- Push notifications (no APNs registration, no Firebase Cloud
  Messaging, no scheduled local notifications).
- Microphone or location services.
- Health, Calendar, or Contacts.
- Bluetooth, motion sensors, or NFC.
- Third-party analytics, advertising, or crash-reporting SDKs (no
  Crashlytics, no Mixpanel, no Sentry, no AdMob, no Facebook SDK).

## Languages

The app's interface is available in English, Traditional Chinese
(繁體中文), and Simplified Chinese (简体中文). The language preference
is stored in your per-user document so it follows you across devices.
This is a UI preference; it does not change anything about what data
is collected.

## What leaves your device, in one table

| Data | Where it goes | Who sees it |
| --- | --- | --- |
| Apple / Google sign-in token | Firebase Auth | Google (Firebase Auth servers) |
| Phone number for SMS code | Firebase Auth | Google + SMS carrier |
| Display name, photo URL, language | Firestore | Google + your family members (display name and photo) |
| Family name, member list | Firestore | Google + your family members |
| Medication list, dose log, pillbox config | Firestore | Google + your family members |
| Medicine-bag photos | Firebase Storage | Google + your family members |
| AI-scan request (photo + prompt) | Google Gemini API | Google (Gemini) |
| Profile avatar | Firebase Storage | Google + any signed-in Uncle's Pills user (avatars are intentionally readable across families) |

The operator (Martin Fu) is the owner of the Firebase project that
holds this data and therefore has administrative access to it in
principle, the same way any Firebase project owner does. We do not
read the data in the normal course of operating the app. We do not
sell, rent, or share it with anyone outside Google's processor role.

## Children

Uncle's Pills is intended for adults managing medications, for themselves or
their family members (including children whom they care for). The app
itself is not directed at children as users. Adults who add a child as
a tracked person in their family are managing that child's data on the
child's behalf. We do not separately collect data from children.

## Your choices

- **Leave a family.** From Settings, you can leave any family you are
  in. Leaving removes your UID from the family's member list. The
  family's medication data continues to exist for the remaining
  members.
- **Delete a family.** An admin can delete an entire family. This
  removes the snapshot document, all medicine-bag photos for that
  family, and the family root document.
- **Sign out.** Sign out from Settings → Account. Your data remains on
  the server; signing back in restores it.
- **Remove your Gemini key.** Settings → Account has a "Clear key"
  button. After this, AI scan is disabled and the app no longer
  contacts Gemini.
- **Delete the app.** Deleting Uncle's Pills from your device removes the
  local cached copy of your data. To also remove the server-side data,
  delete the families you administer first, or contact us at the email
  below for help removing your account.

## Sharing and selling data

We do not share, sell, rent, or transfer your data to anyone outside
of Google (acting as our cloud-platform processor) and Google (acting
as the operator of the Gemini API when you choose to use AI scan). We
do not have advertising partners. We do not buy or enrich data about
you from third parties.

## Changes to this policy

If we change how Uncle's Pills handles your data, we will update this
document and change the "Last updated" date at the top. Material
changes will ship as a new version of the app released through the
App Store.

## Contact

Questions about this policy, or requests related to your data, go to
**unclespills-privacy@fufamily.com**.
