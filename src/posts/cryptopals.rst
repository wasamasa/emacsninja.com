((title . "Cryptopals")
 (date . "2018-03-03 22:46:35 +0100")
 (emacs? . #f))

Solving the `cryptopals crypto challenges`_ was easily the most fun
I've had programming.  If you happen to work on public-facing code
that relies on cryptography, by all means do these challenges.  There
is no crazy math involved [1]_ and the only prerequisite is that
you're familiar with any programming language.  My solutions can be
found on GitHub_ and include notes_ on each exercise, some of which
spoil the puzzle bits.

I've learned the following while completing the original set of 48
exercises:

- You should under no circumstances use ECB as cipher mode
- Padding is a crucial thing to get right, both when attacking
  cryptographic systems and when implementing them
- An attacker can bitflip your ciphertexts into anything they want and
  the only thing you can do about it is checking whether they've been
  tampered with before decrypting (like with a MAC or signature)
- Do not ever reuse a nonce or you'll weaken your crypto drastically
- It's much easier to exploit a sidechannel than attacking the
  cryptographic primitive
- Overly detailed error messages can form a sidechannel
- Do not seed a RNG with the current time, use your system's CSPRNG
  instead
- Do not use MT19937 for cryptographic purposes, given enough
  observation its next values can be predicted
- Do not reuse your key as IV
- Don't invent your own MAC scheme, it may be susceptible to length
  extension attacks
- Even something like a timing leak can form an exploitable
  sidechannel that circumvents the cryptographic system
- Diffie-Hellman is susceptible to MITM attacks
- Make sure to verify the parameters in asymmetric protocols for
  values that make the shared secret predictable and abort when
  encountering one
- Don't do textbook RSA, padding is crucial
- Do not use low exponents with RSA
- Do not use PKCS#1 v1.5 padding with RSA

One more thing that doesn't fit into a short sentence.  You've most
certainly heard the advice "Don't implement your own crypto".  This
advice isn't the whole truth because it doesn't explain what exactly
"your own crypto" means.  Cryptography in software consists of
primitives that are put together to achieve something useful, such as
a hash function and a block cipher to form a HMAC.  These primitives
may be considered safe in isolation, however that doesn't mean their
combination will be equally safe.  These combinations are called
cryptographic systems and the security of one relies upon making sure
none of the invariants are violated.  Therefore, creating your own
cryptosystem out of stock crypto primitives also counts as "your own
crypto" and is rightfully considered dangerous.  Your best bet is to
use a vetted library that has been designed so that it's hard to use
incorrectly, such as libsodium_.

.. _cryptopals crypto challenges: https://www.cryptopals.com/
.. _GitHub: https://github.com/wasamasa/cryptopals
.. _notes: https://github.com/wasamasa/cryptopals/blob/master/notes.md
.. _libsodium: https://github.com/jedisct1/libsodium

.. [1] The challenges like to emphasize that it's only 9th grader
       math.  This is almost correct, you'll want to look up basic
       statistics (which I've had in 12th grade) and modular
       arithmetic (which I've had at college).
