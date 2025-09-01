# Wallet Integration Guide

## Table of Contents

* [Getting Credentials Into the Wallet](#getting-credentials-into-the-wallet)

## Getting Credentials Into the Wallet

### Opening a Deep Link / Universal App Link (From Outside)

Deep links are useful because they allow users to interact with the wallet
from the outside (while they're not in the wallet app itself), but instead
are clicking on a link in a mobile web browser, opening a link pasted to them
in a chat app, or scanning a URL QR code with the general mobile camera.

### Pasting a Deep Link (While In Wallet)

Same as above; you should be able to paste a deep link into the 'Add Credential'
screen textbox, or scan a QR code containing a deep link _while inside the
wallet app_.

### Scanning an Interaction URL (While In Wallet)

Not currently supported, see issue https://github.com/openwallet-foundation-labs/learner-credential-wallet/issues/764

### Pasting Raw VC or VP JSON Into Wallet Textbox

On the 'Add Credential' screen, you can paste the full JSON representation
of a VC that you've copied into your clipboard (such as from a website with VC
examples).

### Scanning Raw VC/VP JSON QR Code (While In Wallet)

(Same as previous item)
While inside the wallet, you can use the 'Scan QR Code' button on the 'Add
Credential' screen to read a QR code that contains (as text) the raw JSON
of a VC or a VP.

### Pasting a URL to a Statically Hosted VC/VP

If a VC or VP is hosted on some website (like github pages or Google Drive),
you can just copy and paste the URL to it into the textbox on the 'Add Credential'
screen.

NOTE: May need the require the use of a CORS proxy (double check on this).

### Scanning a URL to a Statically Hosted VC/VP (While In Wallet)

Same as above, but instead of copying and pasting, you can use the 'Scan QR Code'
button from inside the wallet, and it will fetch and store the VC on the other
end. (Same deal, re CORS proxy)

### Upload VC from File

You can also add a VC you have stored as a file by going to 'Add Credential'
screen and clicking on 'Add from file' button.
Note that this includes both files stored locally on the device, but also
many mobile OSs also add other file storage services (like Google Drive, Dropbox,
etc) to this screen.

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

