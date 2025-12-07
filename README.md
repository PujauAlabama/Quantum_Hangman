# Quantum Hangman

Welcome to **Quantum Hangman!** Game

## Game Rules

- It's a letter guessing game from a word randomly chosen from a list of word (that the player would obviously not be able to see)
- You would be able to guess only one letter at a time and would have at most 6 wrong tries.
- Inially it starts with all dashed lines equal to the length of the word . Everytime you guess the correct letter, it shows up in all dashed spaces for all of it's occurences.
- For each wrong number of tries keeps on decreasing.
- If you would be able to finish guessing the word before 6 wrong tries get over, you win. Otherwise, you lose.
- Beautiful <span style="color:orange; font-size:18px">interface</span>

## Implementation Details
- I have done this two ways, first using Grover's algoritm, second one is Post Quantum Encryption techniques.

### <ins>Method-I : Using Grover's Algoritm </ins>
-  **Grover's Algorithm** is a quantum algorithm that helps to search for an elemnet from unstructured database of (N) entries, which is computationally faster than  any classical algorithm.
- See the [documentation](https://en.wikipedia.org/wiki/Grover%27s_algorithm) for the core idea of this algorithm.
- Firstly, to implemnet hangman game using above, we have to treat each letter of a word as a quantum state.
- As in the hangman game, a player have to guess one letter among 26 english alphabets, we need atleast 26 states. But as the quantum states appears in numbers which are integral powers of 2 (E.g. N= 2,4,8,16,....., because of the binary feature of quantum states taking values either 0 or 1), I am considering 2^5=32 states, which would appear with letter_to_index conversion as 0 to 25 in python code for a to z (I would take .lower() for Capital letter input) and for 26 to 31 indices, I would opt for padding.
- The inherent idea is , if the player gueesed correct letter then the state associated with that letter, would be inverted and amplified using Grover's algorithm. Because of which with proper number iteration of of amplifications, we can get maximum probability of the correctly guessed state and that would be revealed for all it's ocuurences.  But for the wrong guess though the state would be inverted, but there would be no amplification and no word would be revealed.
- The _build_diffuser_circuit() function will be the diffuser part amplifying the correctly gussed state using the reflection of the inverted state about the mean of all states.
- The _backend_grover_sample() function will implement a quantum circuit for every guess and combine the state inverting Oracle part and Diffuser function. Besides that, it runs the quantum circuit and  transforms the states from bit-string binary format to index format using binary to decimal conversion and the registers the individual counts in a dictionary.
- The quantum_measure() function keeps an track on which letters have been chosen, and whether the current guess is correct or not. Then it decides iteration number for diffuser amplification using formula,
  
$$
iter  \approx
\begin{cases}
  \frac{\pi}{4} \times \sqrt{\frac{N}{ M}}, & \text{if } M=1 \\
  0 ,   & \text{if } M=0
\end{cases}
$$

 
  

  where M can value is either 1 (correct guess) or 0 (wrong guess). Then it will calculate the probabilities for individual appearances by their number counts.
- The _print_counts_descending() function would print the individual count and their probability after performing it all in main function, where we can see the maximum probability is coming for the right guess.




###  <ins>Method-II : Using Kyber & AEAD(ChaCha20-Poly1305) </ins>
- For this part I have used ML_KEM, which is a Kyber Key Encapsulation mechanism (ML stands for Module Lattice). Normal encryption procedures used for classical computing (e.g. RSA, ECC, DH) don't really stand well with the application of Quantum Computing (e.g. Shor's algorithm, Grover's algorithm). Kyber uses lattice cryptography based Module-LWE problem, which associates module/rings with higher dimensional space, bringing forth encryption techniques, that is so far Quantum safe.
- See [documentation](https://en.wikipedia.org/wiki/Kyber) for more information about Kyber
- Among the three ML_KEM packages ML_KEM1024 provides the best encryption as it is capable to hold longer size of data each for public key, secret key and ciphertexts. But depending on our usecase, even ML_KEM512 module would do fine.
- The find_ml_kem_module() function would first search for available ML_KEM package, then would use Kyber module (named mod here) to generate public key, secret key, Kyber encryption with public key would give Kyber_Ciphettext(ct_kem) and a shared secret(ss_enc). We get another shared_secret from Kyber wrapper function using ct_kem and secret key. Provided both match we can ahead for the next process, otherwise we would have AEAD failure.
- The derive_aead_key_hkdf() function is implemented using hash based function SHA-256. Apart from SHA-256, HKDF Constructor takes more three inputs, info: It's a combination of a python literal, big-endian byte form of position index of guessed letter and the byte-from of guessed letter. E.g. if Bob is guessing letter "i" from word "spin" , $info=b"letter" + b"||" + b"\x00\x00\x00\x02" + b"||" + b"\x69"$. length would be 32 byte , which is the size of the output , salt: random string that add up before the data, which we have ommitted here. It will give 32 byte key that is compatible for AEAD ChaCha20-Poly1305 algorithm. 
- The aead_encrypt_byte() function takes use of aead_key generated from above function and form an aead constructor. Besides it uses a 12 byte random number nonce and encrypts the letter and returns nonce and aead_Ciphertext. The associated data is a metadata to enhance the level of encryption, which I have ommitted here.
- The aead_decrypt_byte() function uses the aead constructor incorporating the outputs of above function and returns the recovered letter, which would be the decrypted letter and should match with the guess.
- The encapsulate_letter() and decapsulate_and_decrypt_letter() function uses above functions and perform the letter encryption into Ciphertext and then decrypting them with right keys and guess letters (one at a time though).
- The game would be realized as follows: Alice (the host) would be having the access to all the letters of the word as quantum states , therefore would be encrypting them using encapsulate_letter() function and Bob (the player) would be guessing one letter at a time that he would be passing on to Alice and then that would be used in decapsulate_and_decrypt_letter() function to derive recovered letter. The the recovered letter would be matched to guess letter in run_hangman() function, which will match provided the letter exists in the word, as there will already be an encryption for that letter, the letter would be revealed and we can go for next stage of the game. Taking a grong guess of letter would lead to a wrong aead_key, which would give AEAD failure.
  
![PQCrypto Hangman Flowchart](images/hangman_flowchart.png)



See the [documentation](https://example.com/docs).

![Logo](https://example.com/logo.png)

> “This project is awesome!”  
> — Someone Famous

## How it works

```python
print("Hello, World!")
```
