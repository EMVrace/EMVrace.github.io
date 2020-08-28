
EMV, named after its founders Europay, Mastercard, and Visa, is the international protocol standard for smartcard payment. As of December 2019, EMV is used in over 9 billion debit and credit cards worldwide. Despite the standard's advertised security, various issues have been previously uncovered, deriving from logical flaws that are hard to spot in EMV's lengthy and complex specification, running over 2,000 pages.

We present a comprehensive model of EMV, specified in the [Tamarin](https://tamarin-prover.github.io/) verification tool. Using our model, we automatically identified several authentication flaws. One of the encountered flaws, present in the Visa contactless protocol, leads to a **PIN bypass** attack for transactions that are presumably protected by cardholder verification, typically those whose amount is above a local PIN-less upper limit (e.g., currently 80 CHF in Switzerland). This means that your PIN won't prevent criminals from using your Visa contactless card to pay for their transaction, even if the amount is above the mentioned limit. To carry out the attack, the criminals must have access to your card, either by stealing it/finding it if lost, or by holding an NFC-enabled phone near it.

This work will be presented at the [42<sup>nd</sup> IEEE Symposium on
Security and Privacy (S&P 2021)](https://www.ieee-security.org/TC/SP2021/index.html).

## Demonstrating the attacks

To demonstrate how easy it is to exploit the vulnerabilities we found, we developed a proof-of-concept Android application. Our app implements [man-in-the-middle attacks](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) on top of a [relay attack](https://en.wikipedia.org/wiki/Relay_attack) architecture, displayed below.

![Image](relay_attack.png "Relay attack")

The outermost devices are the real payment terminal (on the left) and the victim's contactless card (on the right). The phone near the payment terminal is the attacker's Card emulator device and the phone near the victim's card is the attacker's POS emulator device. The attacker's devices communicate with each other over WiFi, and with the terminal and the card over NFC.

Our app does not require root privileges or any fancy hacks to Android and we have successfully used it on Pixel and Huawei devices.

### Bypassing the PIN for Visa cards

This attack allows criminals to complete a purchase over the PIN-less limit with a victim's contactless card without knowing the card's PIN. The attack consists in a modification of a card-sourced data object --the *Card Transaction Qualifiers*-- before delivering it to the terminal. The modification instructs the terminal that:
1. PIN verification is not required, and
1. the cardholder was verified on the consumer's device (e.g., a smartphone).

<!--The modified CTQ escapes detection because this data object is ***not*** authenticated by the card. -->Technical details can be found in our paper and a video demonstration of the attack for a **200 CHF** transaction is given below.

<div style=" margin: auto; width: 560px;">
<iframe width="560" height="315" src="https://www.youtube-nocookie.com/embed/JyUsMLxCCt8" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

We also tested the attack in live terminals at actual stores. For all of our attack tests, we used our own credit/debit cards; no merchant or any other entities were defrauded.

The EMV contactless protocols (known as *kernels* in EMV's terminology) are Mastercard, Visa, American Express, JCB, Discover, and UnionPay. Out of these, and based on their specification at [EMVCo](https://www.emvco.com/), our PIN bypass attack applies to the Visa, Discover, and UnionPay protocols, but we have not tested the last two in practice.

### Making the terminal accept fake offline transactions

This attack allows a criminal to use their own card to complete a low-value and offline transaction, while not being actually charged. The attack consists in a modification of a card-produced data --the *Transaction Cryptogram*-- before delivering it to the terminal. The terminal cannot detect this modification; only the bank can, yet after the consumer/criminal is long gone with the goods.

This attack applies to both Visa and Mastercard transactions. In the case of the latter, it only applies to transactions with (likely old) cards that do not support the CDA authentication method (see [EMV Book 2 v4.3](https://www.emvco.com/wp-content/uploads/documents/EMV_v4.3_Book_2_Security_and_Key_Management_20120607061923900.pdf)). For ethical reasons, we did not test this second attack in practice.

<!--
## Related work

There exist other (practical) works out there that implement relay as well as other other NFC-related attacks on contactless payment systems. Some of them are:

* [NFCGate](https://github.com/nfcgate): an Android app meant to capture, analyze, or modify NFC traffic. A usage of the app will be presented at [WOOT'20](https://www.usenix.org/conference/woot20/presentation/klee). The app requires root privileges and [Xposed](https://repo.xposed.info/) framework.
* [First contact](https://i.blackhat.com/eu-19/Wednesday/eu-19-Galloway-First-Contact-Vulnerabilities-In-Contactless-Payments-wp.pdf): presented at [Black Hat Europe 2019](https://www.blackhat.com/eu-19/), implements a PIN bypass on Visa cards very similar to ours. The main difference between their attack and ours is that their attack requires also a modification of a terminal-source data object --the *Terminal Transaction Qualifiers*-- before delivering it to the card. Another difference is on implementation: their prototype uses wired Raspberry Pi boards while ours is an innocent-looking phone app that aligns with the current trend of people paying with the phones.
* [Man-in-the-NFC](https://www.slideshare.net/codeblue_jp/man-in-the-nfc-by-haoqi-shan-and-qing-yang): presented at [Defcon 25](https://www.defcon.org/html/defcon-25/dc-25-index.html), implements a relay attack that uses two Software Defined Radio (SDR) boards.
* [EMVemulator](https://github.com/MatusKysel/EMVemulator): implements [Roland and Langer's attack](https://www.usenix.org/conference/woot13/workshop-program/presentation/roland) which combines pre-play and downgrade.
* [NFC Hacking: the easy way](https://www.xinmeow.com/wp-content/uploads/2018/01/DEFCON-20-Lee-NFC-Hacking.pdf): presented at [Defcon 20](https://www.defcon.org/html/defcon-20/dc-20-index.html), uses [NFCProxy](https://sourceforge.net/p/nfcproxy/wiki/Home/) to implement relay using Android phones.
-->

## Acknowledgments

Parts of the code of our app were inspired by the apps [EMVemulator](https://github.com/MatusKysel/EMVemulator), [EMV-Card ROCA-Keytest](https://github.com/johnzweng/android-emv-key-test), and [SwipeYours](https://github.com/dimalinux/SwipeYours). We thank their authors as well as [EFT Lab](https://www.eftlab.com/knowledge-base/145-emv-nfc-tags/) for making EMV's TLV tags and description available.

## About us

We are security researchers with the [Department of Computer Science](http://www.inf.ethz.ch/) at [ETH Zürich](https://www.ethz.ch/en). See our individual pages:
* [Prof. Dr. David Basin](https://people.inf.ethz.ch/basin/)
* [Dr. Ralf Sasse](https://people.inf.ethz.ch/rsasse/)
* [Dr. Jorge Toro](https://jorgetp.github.io)
