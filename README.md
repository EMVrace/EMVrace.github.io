**EMV** (named after its founders Europay, Mastercard, and Visa) is the international protocol standard for in-store smartcard payment. In December 2020, EMVCo [reported](https://www.emvco.com/wp-content/uploads/documents/EMVCo-Annual-Report-2020.pdf) 9.89 billion EMV cards in circulation worldwide. Despite the standard's advertised security, various issues have been previously uncovered, deriving from logical flaws that are hard to spot in EMV's lengthy and complex specification, running over 2,000 pages.

We have specified a comprehensive model of the EMV protocol, using the [Tamarin](https://tamarin-prover.github.io/) model checker. Using our model, we identified several authentication flaws that lead to two critical attacks:

1. Criminals can complete a purchase over the PIN-required limit with a victim's Visa contactless card without knowing the card's PIN. Namely *the PIN in your Visa card is useless* since it won't prevent your card from being used for unauthorized, high-value purchases.

1. Criminals can trick a terminal into transacting with a victim's  Mastercard contactless card while believing it to be a Visa card. This *card brand mixup* attack, in combination with the above PIN bypass for Visa cards, results in a PIN bypass also for Mastercard cards. Following our disclosure, *Mastercard has since implemented defense mechanisms*, which we experimentally confirmed as effective against the attack.

## Demonstrating the attacks

To demonstrate the feasibility of the attacks, we developed a proof-of-concept Android application. Our app implements the attacks as [man-in-the-middle](https://en.wikipedia.org/wiki/Man-in-the-middle_attack) attacks built on top of a [relay attack](https://en.wikipedia.org/wiki/Relay_attack) architecture, using two NFC-enabled phones.

![Image](assets/img/relay_attack.png "Relay attack")

The outermost devices are the payment terminal (on the left) and the victim's contactless card (on the right). The phone near the payment terminal is the attacker's card emulator device and the phone near the victim's card is the attacker's POS emulator device. The attacker's devices communicate with each other over WiFi, and with the terminal and the card over NFC.

For the attacks to work, the criminals must have access to the victim's card, either by stealing it, finding it if lost, or by holding the POS emulator near it if still in the victim's possession. The attacks work by modifying the terminal's commands and the card's responses before delivering them to the corresponding recipient.

Our app does not require root privileges or any hacks to the Android OS. We have used it on Google Pixel, Samsung, and Huawei devices.

### Attack on Visa

The attack consists in a modification of the Card Transaction Qualifiers (CTQ, a card-sourced data object), before delivering it to the terminal. The modification instructs the terminal that:
* PIN verification is not required, and
* the cardholder was verified on the consumer's device (e.g., a smartphone).

We have successfully tested this attack with Visa Credit, Visa Debit, Visa Electron, and V Pay cards, but it may also affect Discover and UnionPay cards. A video demonstration for a **200 CHF** transaction with a Visa card is given below<!-- (used to be available also on [YouTube](https://youtu.be/JyUsMLxCCt8))-->. <!--We also tested the attack in live terminals at actual stores. For all of our attack tests, we used our own credit/debit cards. No merchant or any other entities were defrauded.-->

<div id="demo-visa" class="demo" playsinline>
<!-- <iframe src="https://www.youtube-nocookie.com/embed/JyUsMLxCCt8" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe> -->
<video width="100%" poster="assets/img/Visa200CHF-poster.png" controls>
	<source src="assets/video/Visa200CHF.mp4" type="video/mp4">
	<source src="assets/video/Visa200CHF.webm" type="video/webm">
	Your browser does not support the video tag.
</video>
</div>

{: .pt-3 }
A full technical report of this attack is given in our paper:

<div class="box">
<b>The EMV Standard: Break, Fix, Verify</b><br />
David Basin, Ralf Sasse, and Jorge Toro-Pozo<br />
<em>42nd IEEE Symposium on Security and Privacy (S&P 2021)</em><br />
[<a href="https://ethz.ch/content/dam/ethz/special-interest/infk/inst-infsec/information-security-group-dam/research/publications/pub2021/emv-sp21.pdf">pdf</a>] 
[<a href="https://ethz.ch/content/dam/ethz/special-interest/infk/inst-infsec/information-security-group-dam/research/publications/pub2021/BasinST21sp.bib">bibtex</a>] 
[<a href="https://github.com/EMVrace/EMVerify">model</a>]
</div>

### Attack on Mastercard

This attack primarily consists in the replacement of the card's legitimate Application Identifiers (AIDs) with the Visa AID `A0000000031010` to deceive the terminal into activating the Visa kernel. The attacker then simultaneously performs a Visa transaction with the terminal and a Mastercard transaction with the card. In the Visa transaction, the attacker applies the aforementioned attack on Visa.

For this attack to work, the terminal's authorization request must reach the card-issuing bank. Requirements for this include:
* the terminal does not decline offline even if the card number (PAN) and the AIDs indicate different card brands, and
* the merchant's acquirer routes the transaction authorization request to a payment network that can process Mastercard cards.

We have successfully tested this attack with Mastercard Credit and Maestro cards, but it may also affect JCB and American Express cards. A video demonstration for a **400 CHF** transaction with a Maestro card is given below<!-- (available also on [YouTube](https://youtu.be/8d7UgIiMRBU))-->.

<div id="demo-mastercard" class="demo">
<iframe src="https://www.youtube-nocookie.com/embed/8d7UgIiMRBU" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
<!--<video width="100%" poster="assets/img/Maestro400CHF-poster.png" controls>
	<source src="assets/video/Maestro400CHF.mp4" type="video/mp4">
	<source src="assets/video/Maestro400CHF.webm" type="video/webm">
	Your browser does not support the video tag.
</video>-->
</div>

{: .pt-3 }
A full technical report of this attack is given in our paper:

<div class="box">
<b>Card Brand Mixup Attack: Bypassing the PIN in non-Visa cards by Using Them for Visa Transactions</b><br />
David Basin, Ralf Sasse, and Jorge Toro-Pozo<br />
<em>30th USENIX Security Symposium (2021)</em><br />
[<a href="https://ethz.ch/content/dam/ethz/special-interest/infk/inst-infsec/information-security-group-dam/research/publications/pub2021/sec21fall-basin.pdf">pdf</a>] 
[<a href="https://ethz.ch/content/dam/ethz/special-interest/infk/inst-infsec/information-security-group-dam/research/publications/pub2021/BasinST21uss.bib">bibtex</a>] 
[<a href="https://github.com/EMVrace/EMVerify-PAN-routing">model</a>]
</div>

<!--
### Making the terminal accept fake offline transactions

This attack allows a criminal to use their own card to complete a low-value, offline transaction, while not being actually charged. The attack consists in a modification of a card-produced data object --the Application Cryptogram-- before delivering it to the terminal. The terminal cannot detect this modification; only the bank can, yet after the consumer/criminal is long gone with the goods.

This attack applies to both the Visa and Mastercard protocols. In the case of the latter, it only applies to transactions with (likely old) cards that do not support the CDA authentication method. For ethical reasons, we did not test this second attack in practice.-->

## Media

Our findings have drawn significant media attention (a [Google search](https://www.google.com/search?q=emv+pin+bypass+attack+eth) can give an idea). Below we list some of the most relevant articles available on the web:

<div class="row">
	<div class="col-sm-6">
		<h5>On the attack for Visa</h5>
		<ul>
			<li><a href="https://www.zdnet.com/article/academics-bypass-pins-for-visa-contactless-payments/">ZDNet</a></li>
			<li><a href="https://thehackernews.com/2020/09/emv-payment-card-pin-hacking.html">The Hacker News</a></li>
			<!--<li><a href="https://developer.mastercard.com/blog/multi-layered-security-stops-pin-bypass/">Mastercard developers</a></li>-->
			<li><a href="https://www.srf.ch/news/schweiz/eth-forscher-warnen-sicherheitsluecke-bei-visa-kreditkarten-entdeckt">Schweizer Radio und Fernsehen (SRF)</a></li>
			<li><a href="https://technews.acm.org/archives.cfm?fo=2020-09-sep/sep-04-2020.html#1130993">ACM TechNews</a></li>
			<li><a href="https://www.heise.de/security/meldung/Zahlen-ohne-PIN-Forscher-knacken-Visas-NFC-Bezahlfunktion-4881555.html">heise</a></li>
			<li><a href="https://www.vgtv.no/video/205022/ny-svindelmetode-slik-kan-de-kopiere-kortet-ditt-i-koeen">VG TV</a></li>
			<li><a href="https://ethz.ch/en/news-and-events/eth-news/news/2020/09/outsmarting-the-pin-code.html">ETH Zurich</a></li>
		</ul>
	</div>
	<div class="col-sm-6">
		<h5>On the attack for Mastercard</h5>
		<ul>
			<li><a href="https://thehackernews.com/2021/02/new-hack-lets-attackers-bypass.html">The Hacker News</a></li>
			<li><a href="https://technews.acm.org/archives.cfm?fo=2021-02-feb/feb-26-2021.html#1151729">ACM TechNews</a></li>
 			<li><a href="https://ethz.ch/en/news-and-events/eth-news/news/2021/02/security-flaw-detected-for-the-second-time-in-credit-cards.html">ETH Zurich</a></li>
		</ul>
	</div>
</div>

 
## FAQ

<!--<details>
<summary>What cards are affected by the PIN bypass attack?</summary>
<p>Modern contactless cards that run the Visa protocol, including Visa Credit, Visa Debit, Visa Electron, and V Pay cards. Discover and UnionPay might be affected too.</p>
</details>

<details>
<summary>Why is it possible to bypass the PIN in Visa cards?</summary>
<p>The card does not authenticate the data object that defines the cardholder verification method to be used, thus modifying this object is possible without detection.</p>
</details>

<details>
<summary>Is Mastercard exposed to the PIN bypass attack?</summary>
<p>No. In a Mastercard transaction, the card authenticates the data object that defines the cardholder verification method to be used, thus no modification of this object is possible without detection.</p>
</details>

<details>
<summary>What cards are affected by the offline attack?</summary>
<p>We have not tested this attack in practice and thus we don't know for certain. Based on our analysis, Visa and old Mastercard cards seem to be exposed.</p>
</details>-->

<details>
<summary>What cards are/were affected by these attacks?</summary>
<p>We have successfully bypassed the PIN for Visa Credit, Visa Debit, Visa Electron, V Pay, Mastercard Credit, and Maestro cards. Further EMV cards may be affected but we have no proof of this in the wild.</p>
</details>

<details>
<summary>Has there been any response by the affected companies?</summary>
<p>We have disclosed the attacks to both Visa and Mastercard. As a result of our successful disclosure process with Mastercard, the payment network has since implemented and rolled out defense mechanisms against the attack affecting their cards.</p>
</details>

<details>
<summary>What role did Tamarin play in this research?</summary>
<p>Tamarin is a state-of-the-art verification tool. With it, we analysed the full execution flow of an EMV transaction with unboundedly many executions occurring simultaneously in an adversarial environment, where all messages exchanged between the terminal and the card can be read/blocked/injected. The outcome of this analysis was the identification of the novel attacks we focus here, as well as the rediscovery of existing ones. We also used Tamarin to design and verify (under all adversarial conditions explained above) defenses to all attacks.</p>
</details>

<details>
<summary>There have been many attacks on EMV before, what makes these different?</summary>
<p>Practical attacks reported before are either conspicuous and thus hard to exploit in practice, or do not seem lucrative for criminals due to being possible for low-value purchases only. Our attacks allow criminals to carry out high-value fraudulent transactions and are performed using an app that looks just like a commercial payment app such as Apple Pay or Google Pay, thus evading detection.</p>
</details>

<details>
<summary>What went wrong? How can such problems be avoided in the future?</summary>
<p>Critical data sent by the card during a transaction are not authenticated. Complex systems such as EMV must be analyzed by automated tools, like model checkers. Humans cannot deal with the volume of execution steps and branches a complex system has, and so security breaches are often missed.</p>
</details>

<details>
<summary>Should we protect our cards in a “metal wallet” to prevent them being read remotely?</summary>
<p>This might help. Although you still have problems if they are lost or stolen.</p>
</details>

<details>
<summary>What actions should I as a citizen take now to protect myself?</summary>
<p>Protection measures recommended by banks apply. Block your card immediately upon realization it is lost or stolen. Check your bank statement regularly, and immediately report to your bank whenever you see an unrecognized transaction. Additionally, whenever you are carrying an EMV contactless card, make sure nobody is holding a device near it against your will. Also, be aware of your back pocket.</p>
</details>

<!--<details>
<summary>Do you have follow up plans?</summary>
<p>We plan to further refine our formal model in order to consider even more powerful adversaries. Also, we are open to working with Visa and EMV in implementing the fixes we have proposed as well as in verifying future versions of the standard.</p>
</details>-->

<details>
<summary>Where do I find the Android app?</summary>
<p>Nowhere. We do not make it available.</p>
</details>

<!--
## Acknowledgments

Parts of the code of our app were inspired by the apps [EMVemulator](https://github.com/MatusKysel/EMVemulator), [EMV-Card ROCA-Keytest](https://github.com/johnzweng/android-emv-key-test), and [SwipeYours](https://github.com/dimalinux/SwipeYours). We thank their authors. We also thank [EFT Lab](https://www.eftlab.com/) for making the lists of EMV tags and CA public keys available.-->

## Team

[David Basin](https://www.inf.ethz.ch/personal/basin/), [Ralf Sasse](https://people.inf.ethz.ch/rsasse/), and [Jorge Toro](https://jorgetp.github.io/)<br />
Institute of Information Security<br />
Department of Computer Science<br />
ETH Zurich
