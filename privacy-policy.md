# Uncle's Pills Privacy Policy

**Last updated:** 2026-05-25
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

## AI assistance (Google Gemini) — what we send, to whom, and your consent

Uncle's Pills includes two optional features that send data to **Google's
Gemini API** (Google's generative-AI service):

1. **AI medicine-bag scan.** When you tap "Scan medicine bags", the app
   sends the photos you took (or picked from your library) to Gemini so
   it can extract the medication name, Chinese name, dosage, quantity,
   schedule, shape, color, marking, and storage hints into a structured
   form you can review and edit.
2. **AI medication-name autocomplete.** When you type into a medication
   name field (in the manual add/edit form, or in the AI-scan review
   screen), the app debounces by ~500 ms and then sends the text you
   typed to Gemini to fetch up to 8 suggested medication names and
   brand variants.

### Explicit consent is required

Before either of these features will run, Uncle's Pills shows you a
consent screen that names Google as the recipient, lists exactly what
data is sent, and asks you to tap "Allow". **No photo or typed
medication name is sent to Gemini until you tap "Allow".** If you tap
"Not now", AI scan and AI autocomplete are turned off and the app
never contacts Gemini. You can change your mind any time in
**Settings → AI assistance (Google Gemini)**, which also shows when
you originally allowed it and lets you turn it back off.

### What gets sent

| Action | Data sent to Google Gemini |
| --- | --- |
| You tap "Analyze" on the AI scan modal | The base64-encoded photos you added (which may show a patient's name, clinic name, prescribing doctor, drug name, dosage, instructions, and any other text on the medicine bag) plus a prompt asking Gemini to extract the medication fields |
| You type into a medication-name field | The text you typed (after a ~500 ms debounce), plus a prompt asking Gemini to suggest matching drug names |

The request is sent **directly from your device** over HTTPS to
`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent`,
using **your own Google AI Studio API key** (the app is
"bring-your-own-key" — see below). The request is not routed through
any Uncle's Pills server.

### How the API key is stored

You paste a Google AI Studio API key into Settings → AI assistance.
That key is stored in your per-user Firestore document
(`/users/{uid}.geminiApiKey`) so it follows you to other devices you
sign in on. It is read by the app at request time and attached as the
`key=` query parameter on the call to the Gemini endpoint. The key is
readable and writable only by you under our Firestore rules.

### Who receives the data, and under what terms

The recipient is **Google LLC**, operator of the Gemini API. Google
sees the photo bytes (or typed text) and the prompt because it is the
service performing the inference. Google handles the data under
**Google's privacy policy** (https://policies.google.com/privacy)
and the **Google APIs Terms of Service** that govern your AI Studio
key (https://ai.google.dev/terms). Google offers data protections
that we consider materially equivalent to those described in this
policy; we encourage you to read Google's terms before allowing AI
features.

### What we don't do

Uncle's Pills does not run an AI server of its own, does not log or
cache the photos or typed text on any Uncle's Pills server, and does
not share this data with any other AI vendor. **We do not send the
photo or any of your data to Anthropic, OpenAI, or any other AI
vendor.** The only AI calls the app makes are the two Gemini calls
described here. Disabling consent (or removing your Gemini key) is
sufficient to stop all AI calls — the app falls back to its
on-device medication-name list and to manual medication entry.

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
| AI-scan request (photo + prompt) — only if you have allowed AI assistance and configured a Gemini key | Google Gemini API | Google (Gemini), under Google's privacy policy |
| AI-autocomplete request (typed medication name + prompt) — only if you have allowed AI assistance and configured a Gemini key | Google Gemini API | Google (Gemini), under Google's privacy policy |
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
- **Turn off AI assistance.** Settings → AI assistance (Google Gemini)
  has a "Turn off AI features" button. After this, AI scan and AI
  autocomplete are disabled and the app no longer contacts Gemini —
  regardless of whether a Gemini key is configured.
- **Remove your Gemini key.** Settings → AI assistance also has a
  "Remove key" button. With no key configured, the app cannot call
  Gemini even if consent is on.
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
