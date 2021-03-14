### TipAddr: A Short Address Format for Bitcoin Cash's Active Accounts

Sometimes we want to share a bitcoin cash's address on social media (twitter, reddit, etc) such that others can easily send tips to us.

If the address is too long, it will hurt user experience, since a tweet is only 140 characters.

Here we describe a new address format named TipAddr, which is as short as six Chinese characters: "BCH〇〇〇〇"，where "〇〇〇〇" are place holders for four Chinese characters.

To generate such an address, we need to convert an active account which ever had some event on bitcoin cash, into a 48-bit short id. By scanning all the Bitcoin Cash's historical transactions, we can generate a dictionary mapping from 48-bit short ids to P2PKH addresses. The pesudo code is as follow:

```python

id2addr = {}
addr2id = {}
for block in all_blocks:
	for tx in block.transactions:
		for out in tx.outputs:
			if not out.is_P2PKH():
				continue
			bits160 = out.get_20bytes_address()
			for i in range(16):
				shifted_bits = bits160 >> (i*7)
				low_bits = shifted_bits.get_lowest_48_bits()
				if low_bits not in id2addr:
					id2addr[low_bits] = bits160
					addr2id[bits160] = id
					break

```

A 48-bit short id can be converted to four Chinese characters, using following algorithm:

```javascript
function i2char(i) {
	if(i<=6582) {
		return String.fromCharCode(i+0x3400); // Extended Chinese Characters A
	} else {
		return String.fromCharCode(i+0x4E00); // Basic Chinese Characters
	}
}

function short_id_to_tip_address(i) {
	var cksum = i % 65521; // 65521 is the largest 16-bit prime number
	var e0 = (i & 0xFFF);
	var c0 = (cksum & 7);
	e0 = (e0<<3) | c0;
	var e1 = ((i>>12) & 0xFFF);
	var c1 = ((cksum>>3) & 7);
	e1 = (e1<<3) | c1;
	var e2 = ((i>>24) & 0xFFF);
	var c2 = ((cksum>>6) & 7);
	e2 = (e2<<3) | c2;
	var e3 = ((i>>36) & 0xFFF);
	var c3 = ((cksum>>9) & 7);
	e3 = (e3<<3) | c3;

	return "BCH"+i2char(e0)+i2char(e1)+i2char(e2)+i2char(e3)
}
```

Each character contains 12 bits of payload and 3 bits of checksum. Mixing of payload and checksum makes it very hard to generate vanity addresses which are meaningful text in Chinese.

For example, if the short id is 107074494162278, then the tip address would be "BCH礷䰱㗎肵".

Please note that tip addresses are determined by a block chain's history. What's why we must keep the "BCH" prefix. On Bitcoin or Bitcoin SV, "礷䰱㗎肵" may correspond to a 20-byte P2PKH address which is different from the one on Bitcoin Cash. And if a P2PKH address has never occured in block chain's history, it has no corresponding tip address at all.

How are such tip addresses used? If you want to get donations or tips from others, you can include your tip address in a tweet or a hash tag (something like #SupportAlice-BCH礷䰱㗎肵). Other people retweet your tweet or include this hash tag inside their tweets, such that more and more people can see your tip address and your story, among whom some generous persons will eventually send BCH to your tip address.

A browser plugin can help your generate and "register" tip addresses by sending transactions. It can also identify the tip addresses in web pages such that you can easily donate BCH to these addresses by a single click.

