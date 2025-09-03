# Wallet Integration Guide

## Table of Contents

* [Getting Credentials Into the Wallet](#getting-credentials-into-the-wallet)

## Getting Credentials Into the Wallet

### Opening a Deep Link / Universal App Link (From Outside)

Deep links are useful because they allow users to interact with the wallet
from the outside (while they're not in the wallet app itself), but instead
are clicking on a link in a mobile web browser, opening a link pasted to them
in a chat app, or scanning a URL QR code with the general mobile camera.

**Dev note:**
The custom protocols for deep links as well as universal app links are configured
in `app.conf.js`: 

* in the `LinkConfig.schemes` constant
* but also in `expo.ios.associatedDomains` and `CFBundleURLSchemes`
* also in `expo.android.intentFilters`

see https://docs.expo.dev/linking/into-your-app/
and https://docs.expo.dev/linking/overview/#universal-linking

The actual link router is set up in `/app/lib/deepLink.ts > deepLinkConfigFor()`

### Pasting a Deep Link (While In Wallet)

Same as above; you should be able to paste a deep link into the 'Add Credential'
screen textbox, or scan a QR code containing a deep link _while inside the
wallet app_.

**Dev note:** See `/app/screens/AddScreen/AddScreen.tsx > addCredentialsFrom()`
for how deep links are handled.

### Scanning an Interaction URL (While In Wallet)

Not currently supported, see issue https://github.com/openwallet-foundation-labs/learner-credential-wallet/issues/764

**Dev note:** Support for this would have to be added to
`/app/screens/AddScreen/AddScreen.tsx > onReadQRCode()`

### Pasting Raw VC or VP JSON Into Wallet Textbox

On the 'Add Credential' screen, you can paste the full JSON representation
of a VC that you've copied into your clipboard (such as from a website with VC
examples).

**Dev note:** See `/app/screens/AddScreen/AddScreen.tsx > addCredentialsFrom()`
for how deep links are handled.

### Scanning Raw VC/VP JSON QR Code (While In Wallet)

(Same as previous item)
While inside the wallet, you can use the 'Scan QR Code' button on the 'Add
Credential' screen to read a QR code that contains (as text) the raw JSON
of a VC or a VP.

**Dev note:** See `/app/screens/AddScreen/AddScreen.tsx > onReadQRCode()`
which calls `/app/lib/decode.ts > credentialsFrom`

### Pasting a URL to a Statically Hosted VC/VP

If a VC or VP is hosted on some website (like github pages or Google Drive),
you can just copy and paste the URL to it into the textbox on the 'Add Credential'
screen.

**Dev note:** See `/app/screens/AddScreen/AddScreen.tsx > addCredentialsFrom()`
which calls `/app/lib/decode.ts > credentialsFrom`

TODO: Add the ability to fetch URLs through a CORS proxy.

### Scanning a URL to a Statically Hosted VC/VP (While In Wallet)

Same as above, but instead of copying and pasting, you can use the 'Scan QR Code'
button from inside the wallet, and it will fetch and store the VC on the other
end. (Same deal, re CORS proxy)

**Dev note:** See `/app/screens/AddScreen/AddScreen.tsx > onReadQRCode()`
which calls `/app/lib/decode.ts > credentialsFrom`

TODO: This does mean that scanning a deep link (while inside the wallet)
is not currently supported.

### Upload VC from File

You can also add a VC you have stored as a file by going to 'Add Credential'
screen and clicking on 'Add from file' button.
Note that this includes both files stored locally on the device, but also
many mobile OSs also add other file storage services (like Google Drive, Dropbox,
etc) to this screen.

**Dev note:** See `/app/screens/AddScreen/AddScreen.tsx > addFromFile()`
for more details.

### Restore VCs from Backup

If you have a previous wallet backup (from the Settings > Backup Wallet screen),
if you go to Settings > Restore wallet, any VCs from that backup (that don't
already exist in your wallet) are also added.

### Receive a Share Intent from Another App

NOTE: We used to support this in previous LCW versions, but it was disabled
due to extra complexity, and no apps were sharing VCs at the time.

for iOS, see:
* ShareKit and App Extensions https://developer.apple.com/library/archive/documentation/General/Conceptual/ExtensibilityPG/index.html
* App Intents https://developer.apple.com/documentation/appintents/

for Android, see:
* Sharesheet docs https://developer.android.com/training/sharing/send

## Example Requests

### Request a VC

```
https://lcw.app/request?request=%7B%22credentialRequestOrigin%22:%22https://interop-alliance.github.io/wallet-to-webapp-demo%22,%22protocols%22:%7B%22vcapi%22:%22https://verifierplus.org/api/exchanges/42cb7245-a732-409f-8182-1db416a793f7%22%7D%7D
```

```json
{
  "credentialRequestOrigin": "https://interop-alliance.github.io/wallet-to-webapp-demo",
  "protocols": {
    "vcapi": "https://verifierplus.org/api/exchanges/42cb7245-a732-409f-8182-1db416a793f7"
  }
}
```

leads to: Exchange Credentials Request popup.
"An organization would like to exchange credentials with you."

Select credentials, wallet POSTS a VP to the interact endpoint.
