
EMV is the international protocol standard for smartcard payment and runs used in over 9 billion cards worldwide, as of December 2019. Despite the standard's advertised security, various issues have been previously uncovered, deriving from logical flaws that are hard to spot in EMV's lengthy and complex specification, running over 2,000 pages.

Our paper ***The EMV Standard: Break, Fix, Verify***, accepted for the [IEEE S&P 2021](https://www.ieee-security.org/TC/SP2021/index.html) symposium, presents a comprehensive model of EMV specified in the [Tamarin](https://tamarin-prover.github.io/) verification tool. Using our model, we automatically identified several authentication flaws. One of the encountered flaws, present in the Visa contactless protocol, leads to a PIN bypass attack for transactions that are presumably protected by cardholder verification, typically those whose amount is above a local upper limit. This means that your PIN won't prevent criminals from using your Visa card to pay for their high-value transactions. To carry out the attack, the criminals must have access to your card, either by stealing it or finding it if lost, or by holding an NFC-enabled phone near it.

A preprint version of the paper is available [here](https://arxiv.org/abs/2006.08249) and the full set of Tamarin models and proofs are at [https://github.com/EMVrace/EMVerify](https://github.com/EMVrace/EMVerify). To interpret/read our models and proofs, background on Tamarin is needed. .

<!--Disclaimer: The information presented in this page as well as in our paper is merely for research. **Do NOT use it for criminal purposes**.-->

## Proving the Attacks

To prove the practical connotation that the vulnerabilities we found have, we developed a proof-of-concept Android app. Our app implements two [man-in-the-middle attacks](https://en.wikipedia.org/wiki/Man-in-the-middle_attack), built on top of a [relay attack](https://en.wikipedia.org/wiki/Relay_attack) architecture, see below.

![Image](relay_attack.png "Relay attack")

The outermost devices are the real payment terminal (on the left) and the victim's contactless card (on the right). The phone near the payment terminal is the attacker's Card emulator and the phone near the victim's card is the attacker's POS emulator. The attacker's devices occur over WiFi with a TCP socket server-client channel. Our app does not require root privileges or any fancy hacks to Android and we have successfully used it on Pixel and Huawei devices.

### Bypassing the PIN for Visa cards

This attack allows criminals to complete a purchase with a victim's contactless card without knowing the card's PIN. The attack consists simply in a modification of a card-sourced data object (called the Card Transaction Qualifiers), before delivering it to the terminal. Our modification tells the terminal that:
1. Online PIN verification is not required, and
1. Consumer Device Cardholder Verification (CDCVM) was performed.

The modified CTQ escapes detection because this data object is ***not*** authenticated by the card. Technical details can be found in our paper.

<!-- and a video demonstration of our PIN bypass for a 200 CHF transaction is given next.

<div style=" margin: auto; width: 560px;height: 315px;">
</div>-->

We also tested the attack in live terminals at actual stores. For all of our attack tests, we always used our own credit/debit cards; no merchant or any other entities were defrauded.

As for Mastercard transactions, this attack does not apply because the card authenticates its messages related to cardholder verification, so tampering with those messages will result in a declined transaction.

### Making the terminal accept fake offline transactions

This attack, which is much less critical that the previous one, allows a criminal to use their own card to complete a low-amount, offline transaction, while not being actually charged. To carry out this attack, the man-in-the-middle modifies the card-produced Transaction Cryptogram (TC). The terminal cannot detect this modification; only the bank can, yet after the consumer/criminal is long gone with the goods. For ethical reasons, we did not test this second attack in practice.

## Related work

There exist other (practical) works out there that implement relay as well as other other NFC-related attacks on contactless payment systems. Some of them are:

* [First contact](https://i.blackhat.com/eu-19/Wednesday/eu-19-Galloway-First-Contact-Vulnerabilities-In-Contactless-Payments-wp.pdf): Galloway and Yunusov presented at [Black Hat Europe 2019](https://www.blackhat.com/eu-19/) a PIN bypass on Visa cards very similar to ours. The main difference between their attack and ours is that their attack requires also a modification of a terminal-source data object (called the Transaction Qualifiers) before delivering it to the card.

* [NFCGate](https://github.com/nfcgate): implements relay, clone cards, and more. It requires root privileges and [Xposed](https://repo.xposed.info/) framework in at least one of the devices.

* [EMVemulator](https://github.com/MatusKysel/EMVemulator): implements [Roland and Langer's attack](https://www.usenix.org/conference/woot13/workshop-program/presentation/roland) which combines pre-play and downgrade.

## Acknowledgments

Parts of the code of our app were inspired by the apps listed above as well as [EMV-Card ROCA-Keytest](https://github.com/johnzweng/android-emv-key-test) (useful for the PKI stuff) and [SwipeYours](https://github.com/dimalinux/SwipeYours) (useful for the APDU stuff), so we thank their authors.

We also thank [EFT Lab](https://www.eftlab.com/knowledge-base/145-emv-nfc-tags/) for making the EMV tags and their description available.

## About us

We are researchers within the [Department of Computer Science](http://www.inf.ethz.ch/) at [ETH Zürich](https://www.ethz.ch/en). You can reach out individually at:
* [Jorge Toro](https://jorgetp.github.io)
* [David Basin](https://people.inf.ethz.ch/basin/)
* [Ralf Sasse](https://people.inf.ethz.ch/rsasse/)

