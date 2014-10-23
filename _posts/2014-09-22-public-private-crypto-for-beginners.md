---
layout: post_page
title: Public/Private Key Cryptography for Beginners
tags: Cryptography, Asymmetric, Beginners
---

_Disclaimer:_ I am not even close to being a cryptography expert. All the details here are the limits of my knowledge and could very well be wrong. Please tweet [@darkphnx](https://twitter.com/darkphnx) with any corrections.

Public/Private key encryption is the cornerstone of secure communication across the internet, it underpins both HTTPS and SSH. By the end of this post you should have a rough idea of how asymmetric encryption works, and it's practical application in common technologies. We won't be looking at the maths behind the technology, for an introduction to that you should turn to [Simon Singh's Code  Book][1].

Lets start with a quick look at symmetric encryption and it's problems, then we'll take a look at how asymmetric encryption solves these problems.

## Symmetric Encryption
Symmetric encryption allows you to encrypt data using a shared encryption key (such as a password) that both parties know. This same key is used to decrypt data by the recipient. Examples of symmetric algorithms include [AES, DES and Blowfish][2] .

Symmetric encryption presents a couple of challenges; the big one is initial key sharing. How do you get the passphrase from your sender to your recipient via a secure means? You can't send an email, an SMS or any other plain text message, or your key could become compromised. You can't use a telephone because someone might be listening in. Realistically, the only way to securely share a passphrase is to meet in person and exchange the key. Of course, if your key then becomes compromised by some other means in the future, you'll have to meet again to exchange a new key.

## Asymmetric Encryption
Asymmetric encryption (also known as public-key encryption) sovles the key exchange problem by building it's keys as two parts, a public key and a private key. The public part of the key is (as the name suggests) public and can be safely published. The private part of the key must be kept secret by it's owner. If you want to send a secure message to another person, all you need is their public key, which you could have obtained via their website, or some other means such as e-maill. Since public keys don't need to be secret the method of transmission is unimportant. Popular asymmetric encryption algorithms include [RSA and DSS][3].

When you want to send a message to someone, you get their **public key** and encrypt your message against that key. The generated message will only be decodable by their **private key**. Even you will not be able to decrypt that message. When they want to send you a message, they must fetch your public key, and encrypt their message against that, which will generate a message that only you can decrypt.

The drawback is each person in the conversation will require their own public/private keypair to facilitate two-way secure communications. Each person will also require a program to generate you a public/private keypair.

![Asymmetric Encryption](/assets/asymmetric.png)

## Signing Messages
Public-key encryption also gives you the oppourtunity to verify the authenticity of the sender of a message. Using a private key, it is possible to sign a message which can be verified by the recipient with the public key of the sender. This gives you an extra layer in security, knowing that the message has been sent from a trusted source.

[1]: http://www.amazon.co.uk/gp/product/1857028899/ref=as_li_tl?ie=UTF8&camp=1634&creative=6738&creativeASIN=1857028899&linkCode=as2&tag=danwent-21&linkId=VQZQD7DEECEH4NWU "amazon.co.uk - Simon Singh's Code Book"
[2]: http://en.wikipedia.org/wiki/Symmetric-key_algorithm "Symmetric Key Algorithm - Wikipedia"
[3]: http://en.wikipedia.org/wiki/Public-key_cryptography "Public-key Cryptography - Wikipedia"