<link rel="stylesheet" href="assets/css/custom.css">

# MapleCTF-Writeups

This weekend from September 29th to October 1st, I competed in the MapleCTF competition as a part of the [49th Security Division](https://49sd.com)! Our team name was "stack smashing slackers."

This competition was really fun! Our team ended up in 38th place with 825 points! I personally solved 4 challenges (and submitted the challenge flag for the survey, but there's not much there for a writeup), earning 725 points for my team!

Thank you so much to the Maple Bacon team from University of British Columbia for hosting this competition! I hope to compete next year too!

# Misc: A Quantum Symphony

**Points:** 425

**Author:** hiswui

**Description:** On what seemed like an ordinary day of lazing around on your couch, you were suddenly kidnapped by the US intelligence agencies. They claim that you know the secret song that prevents the impending Quantum apocalypse. The answer seems to lie with your kindergarten teacher Ms. Matter Eviller. You being to recall a strange song that has been stuck in your head since childhood...

`nc quantum-symphony.ctf.maplebacon.org 1337`

**Files:** [dist.zip](files/quantum/dist[1])

## Writeup

This was probably my favorite challenge of those I completed! It was really fun, involving quantum computing and a bit of music theory!

First, in the dist.zip file, there is a file, server.py. It includes these contents:

```py
#!/usr/bin/env python3

from qiskit import QuantumCircuit
import qiskit.quantum_info as qi
from base64 import b64decode
from secret import CHORD_PROGRESSION, FLAG, LORE

QUBITS = 4

# No flat notes because they're just enharmonic to the sharps
NOTE_MAP = {
    '0000': 'C',
    '0001': 'C#',
    '0010': 'D',
    '0011': 'D#',
    '0100': 'E',
    '0101': 'F',
    '0110': 'F#',
    '0111': 'G',
    '1000': 'G#',
    '1001': 'A',
    '1010': 'A#',
    '1011': 'B',
    '1100': 'X', # misc note (aka ignore these)
    '1101': 'X',
    '1110': 'X',
    '1111': 'X',
}

# legend says that one only requires the major and minor chords to unlock the secret gates
CHORD_MAP = {
    'Cmaj' : {'C', 'E', 'G'},
    'C#maj': {'C#', 'E#', 'G#'},
    'Dmaj' : {'D', 'F#', 'A'},
    'D#maj': {'D#', 'G', 'A#'},
    'Emaj' : {'E', 'G#', 'B'},
    'Fmaj' : {'F', 'A', 'C'},
    'Gmaj' : {'G', 'B', 'D'},
    'G#maj': {'G#', 'C', 'D#'}, 
    'Amaj' : {'A', 'C#', 'E'},
    'A#maj': {'A#', 'D', 'F'},
    'Bmaj' : {'N', 'D', 'F#'},

    'Cmin' : {'C', 'D#', 'G'},
    'C#min': {'C#', 'E', 'G#'},
    'Dmin' : {'D', 'F', 'A'},
    'D#min': {'D#', 'F#', 'A#'},
    'Emin' : {'E', 'G', 'B'},
    'Fmin' : {'F', 'G#', 'C'},
    'F#min': {'F#', 'A', 'C#'}, 
    'Gmin' : {'G', 'A#', 'D'},
    'G#min': {'G#', 'B', 'D#'},
    'Amin' : {'A', 'C', 'E'},
    'A#min': {'A#', 'C#', 'F'},
    'Bmin' : {'B', 'D', 'F#'}
}


def get_chord_from_sv(state_vector) -> str:
    '''
    Returns the correct chord from the CHORD_MAP corresponding to the superposition of the 
    '''
    state_to_probability_map = [(bin(i)[2:].rjust(QUBITS, '0'), state_vector[i].real ** 2) for i in range(len(state_vector))]
    state_to_probability_map.sort(key=lambda pair: pair[1], reverse=True)
    notes = set()

    for i in range(3):
        qubit_state, probability = state_to_probability_map[i]
        notes.add(NOTE_MAP[qubit_state])

    if notes in CHORD_MAP.values():
        for chord in CHORD_MAP.keys():
            if notes == CHORD_MAP[chord]:
                return chord
    return 'INVALID_CHORD'


def get_circuit():
    try:
        qasm_str = b64decode(input("\nThe device demands an OpenQASM string encoded in base 64 from you: ")).decode()
    except:
        print("The device makes some angry noises. Perhaps there was an error decoding b64!")
        exit(0)
    try:
        circ = QuantumCircuit.from_qasm_str(qasm_str)
        circ.remove_final_measurements(inplace=True)
    except:
        print("The device looks sick from your Circuit. It advises you to use the Qiskit Transpiler in order to decompose your circuit into the basis gates (Rx. Ry, Rz, CNOT)")
        exit(0)
    if circ.num_qubits != QUBITS:
        print(f"Your quantum circuit acts on {circ.num_qubits} instead of {QUBITS} qubits!")
        exit(0)
    return circ


def main():
    print(LORE)  
    for chord in CHORD_PROGRESSION:
        circ = get_circuit()
        sv = qi.Statevector.from_instruction(circ)

        # Note from an anonymous NSA cryptanalyst: 
        # I wish there was some way I could convert a zero state vector to any arbitrary state vector of my choosing,,, I feel like I'm so close.
        input_chord = get_chord_from_sv(sv)
        if input_chord != chord:
            print("You entered the wrong chord")
            print(f"the chord you entered: {input_chord}")
            print('Good bye!')
            exit(0)
        print("Gears start shifting...")
        print("I think you entered the correct chord!")

    print("Woah... The device is shaking..,")
    print("The device has been unlocked.")
    print("you see a piece of paper hidden away inside, securely between the gears.")
    print("You pick it up and begin to read it:")
    print(FLAG)

if __name__ == "__main__":
    main()


```

Some important things to note with this code are that it reads user input in the form of OpenQASM strings encoded in base64 to then check if a chord is a correct chord in a progression. It does this by checking the first three highest-probability states of the quantum computer and comparing these with states assigned to musical notes.

In quantum computing, rather than there being a single state at a given time, "superpositions" are used where the computer has multiple states at the same time, but different probabilities of each state being measured. A state is any combination of the values of the qubits, such as 0000 or 0001 or 1010 or anything like that! The combination of all state probabilities in the quantum computer is the state-vector. The state-vector stores a square-root version of the probability to allow for different "phases," but that doesn't matter for the purpose of this challenge, since as we can see in the code, only the real part is squared to get the probability. Other solutions might be possible without this, or possibly using this, since the quantum computer still processes the imaginary numbers, just they aren't registered by the server code!

Some common quantum logic gates (similar to AND, NOT, OR, etc., but for quantum purposes) are the Hadamard gate (H), Pauli-X gate (X), controlled NOT gate (CNOT/CX), Toffoli gate or controlled-controlled-NOT (CCNOT/CCX), and the controlled Hadamard gate (CH). A Hadamard gate creates a superposition, giving the current qubit a 50% chance of being 0 and 50% chance of being 1. Another interesting feature of the gate is that applying it twice cancels it out! The Pauli-X gate simply flips the qubit, making it 1 if originally 0 or 0 if originally 1, similar to the classical NOT gate. Controlled gates like CNOT or CH apply a given gate, such as Pauli-X or Hadamard, respectively, only if another qubit is 1. And CCNOT applies the Pauli-X gate only if two qubits are 1, similar to a classical AND gate. Since OR can be implemented with NOT and AND, there is no need for an elementary equivalent in quantum computing.

For this challenge, we need to convert given states to OpenQASM quantum instructions! For example, for the D minor chord, we need some sort of superposition between D (0010), F (0101), and A (1001). I personally did the C major chord first while solving it, but this is one of the more complicated ones, so it's more useful to explain! My C major implementation was also a bit over-complicated and could be done with just three gates...

Anyways, let's try D minor!

First, for QASM, we need a header to include the standard library and set the quantum computer to 4 qubits!

```qasm
include "qelib1.inc";
qreg q[4];
```

Now, the first qubit on the left (qubit 3) can be 1 or 0, and if it is 1, so is qubit 0 on the far right. Combined, these would give the note of A! We can set a superposition for qubit 3 with the Hadamard gate applied to it and then use CNOT/CX to set qubit 0 to 1 if qubit 3 is 1. This means the computer can either be in state 0000 or 1001, each with a 50% chance.

```qasm
h q[3];
cx q[3], q[0];
```

If qubit 3 is 0, qubit 2 can be 1 or 0 as well! And if it is 1, so is qubit 0, creating the note of F! Now, we need a sort of "if-not"... Well, the Pauli-X gate works just like a NOT and CH applies the Hadamard gate with a condition! Since we want a superposition, we need this rather than CNOT. So we can flip qubit 3 with an X gate, apply CH to 3 and 2, and flip back qubit 3 later with another X gate! We won't do this yet, though, since we still need this qubit for another "if-not" later! And since we need qubit 0 to match qubit 2, we can use another CNOT/CX applied to 2 and 0! Once we flip back qubit 3, this will give a 50% chance of 1001 and a 25% each for 0000 and 0101. 

```qasm
x q[3];
ch q[3], q[2];
cx q[2], q[0];
```

Now, we need to set qubit 1 (second from right) to 1 if both qubits 3 and 2 are 0. We can do the same "if-not" by flipping with a Pauli-X gate first--and luckily we still have qubit 3 flipped from earlier--but now with a CCNOT! In classical terms, this sets qubit 1 to `NOT(q[3]) AND NOT(q[2])` which is exactly what we want! And we can end by flipping back qubits 2 and 3 to their previous states! This finishes our code with a 50% chance of 1001 and a 25% chance of each 0101 and 0010, which fits our chord!

```qasm
x q[2];
ccx q[3], q[2], q[1];
x q[2];
x q[3];
```


Similar logic can be used to generate QASM code for any combination of notes! And theoretically, the other states don't actually have to be 0 probability like I did here, as long as they are not the first three highest probabilities. It's easiest to control this by only allowing three notes to have any probability, but any method of controlling the most likely qubits would work!

Here are the QASM code and base64-encoded versions for all of the chords included. However, C# major and B major both have errors in them... C# major includes a non-existent E# (it would just be F) and B major looks for N instead of B. So we can simply ignore these since they won't show up at all during the challenge! Anyways, here's the code!

C major:

```qasm
include "qelib1.inc";
qreg q[4];

h q[2];
h q[0];
cx q[0], q[1];
ch q[0], q[2];
cx q[0], q[2];
```

Base64-encoded: `aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVsyXTsKaCBxWzBdOwpjeCBxWzBdLCBxWzFdOwpjaCBxWzBdLCBxWzJdOwpjeCBxWzBdLCBxWzJdOw==`

D major:

```qasm
include "qelib1.inc";
qreg q[4];

h q[1];
x q[1];
cx q[1], q[0];
cx q[1], q[3];
x q[1];
ch q[1], q[2];
```

Base64-encoded: `aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVsxXTsKeCBxWzFdOwpjeCBxWzFdLCBxWzBdOwpjeCBxWzFdLCBxWzNdOwp4IHFbMV07CmNoIHFbMV0sIHFbMl07`

D# major:

```qasm
include "qelib1.inc";
qreg q[4];

x q[1];
h q[0];
ch q[0], q[2];
x q[0];
cx q[0], q[3];
x q[0];
```

Base64-encoded: `aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCnggcVsxXTsKaCBxWzBdOwpjaCBxWzBdLCBxWzJdOwp4IHFbMF07CmN4IHFbMF0sIHFbM107CnggcVswXTs=`

E major:

```qasm
include "qelib1.inc";
qreg q[4];

h q[3];
ch q[3], q[0];
cx q[0], q[1];
x q[3];
cx q[3], q[2];
x q[3];
```

Base64-encoded: `aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVszXTsKY2ggcVszXSwgcVswXTsKY3ggcVswXSwgcVsxXTsKeCBxWzNdOwpjeCBxWzNdLCBxWzJdOwp4IHFbM107`

F major:

```qasm
include "qelib1.inc";
qreg q[4];

h q[3];
x q[3];
ch q[3], q[2];
x q[3];
cx q[3], q[0];
cx q[2], q[0];
```

Base64-encoded: `aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVszXTsKeCBxWzNdOwpjaCBxWzNdLCBxWzJdOwp4IHFbM107CmN4IHFbM10sIHFbMF07CmN4IHFbMl0sIHFbMF07`

G major:

```qasm
include "qelib1.inc";
qreg q[4];

x q[1];
h q[3];
x q[3];
ch q[3], q[2];
x q[3];
cx q[3], q[0];
cx q[2], q[0];
```

Base64-encoded: `aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCnggcVsxXTsKaCBxWzNdOwp4IHFbM107CmNoIHFbM10sIHFbMl07CnggcVszXTsKY3ggcVszXSwgcVswXTsKY3ggcVsyXSwgcVswXTs=`

G# major:

```qasm
include "qelib1.inc";
qreg q[4];

h q[3];
x q[3];
ch q[3], q[0];
x q[3];
cx q[0], q[1];
```

Base64-encoded: `aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVszXTsKeCBxWzNdOwpjaCBxWzNdLCBxWzBdOwp4IHFbM107CmN4IHFbMF0sIHFbMV07`

A major:

```qasm
include "qelib1.inc";
qreg q[4];

h q[2];
x q[2];
cx q[2], q[0];
ch q[2], q[3];
x q[2];

```

Base64-encoded: `aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVsyXTsKeCBxWzJdOwpjeCBxWzJdLCBxWzBdOwpjaCBxWzJdLCBxWzNdOwp4IHFbMl07Cg==`

A# major:

```qasm
include "qelib1.inc";
qreg q[4];

h q[2];
cx q[2], q[0];
x q[2];
cx q[2], q[1];
ch q[2], q[3];
x q[2];

```

Base64-encoded: `aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVsyXTsKY3ggcVsyXSwgcVswXTsKeCBxWzJdOwpjeCBxWzJdLCBxWzFdOwpjaCBxWzJdLCBxWzNdOwp4IHFbMl07Cg==`

C minor:

```qasm
include "qelib1.inc";
qreg q[4];

h q[0];
cx q[0], q[1];
ch q[0], q[2];
```

Base64-encoded: `aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVswXTsKY3ggcVswXSwgcVsxXTsKY2ggcVswXSwgcVsyXTs=`

C# minor:

```qasm
include "qelib1.inc";
qreg q[4];

h q[3];
x q[3];
ch q[3], q[2];
x q[2];
ccx q[3], q[2], q[0];
x q[2];
x q[3];
```

Base64-encoded: `aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVszXTsKeCBxWzNdOwpjaCBxWzNdLCBxWzJdOwp4IHFbMl07CmNjeCBxWzNdLCBxWzJdLCBxWzBdOwp4IHFbMl07CnggcVszXTs=`

D minor:

```qasm
include "qelib1.inc";
qreg q[4];

h q[3];
cx q[3], q[0];
x q[3];
ch q[3], q[2];
cx q[2], q[0];
x q[2];
ccx q[3], q[2], q[1];
x q[2];
x q[3];
```

Base64-encoded: `aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVszXTsKY3ggcVszXSwgcVswXTsKeCBxWzNdOwpjaCBxWzNdLCBxWzJdOwpjeCBxWzJdLCBxWzBdOwp4IHFbMl07CmNjeCBxWzNdLCBxWzJdLCBxWzFdOwp4IHFbMl07CnggcVszXTs=`

D# minor:

```qasm
include "qelib1.inc";
qreg q[4];

x q[1];
h q[3];
x q[3];
ch q[3], q[2];
x q[2];
ccx q[3], q[2], q[0];
x q[2];
x q[3];
```

Base64-encoded: `aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCnggcVsxXTsKaCBxWzNdOwp4IHFbM107CmNoIHFbM10sIHFbMl07CnggcVsyXTsKY2N4IHFbM10sIHFbMl0sIHFbMF07CnggcVsyXTsKeCBxWzNdOw==`

E minor:

```qasm
include "qelib1.inc";
qreg q[4];

h q[3];
x q[3];
cx q[3], q[2];
x q[3];
ch q[2], q[1];
cx q[3], q[1];
cx q[1], q[0];

```

Base64-encoded: `aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVszXTsKeCBxWzNdOwpjeCBxWzNdLCBxWzJdOwp4IHFbM107CmNoIHFbMl0sIHFbMV07CmN4IHFbM10sIHFbMV07CmN4IHFbMV0sIHFbMF07Cg==`

F minor:

```qasm
include "qelib1.inc";
qreg q[4];

h q[3];
x q[3];
ch q[3], q[2];
cx q[2], q[0];
x q[3];

```

Base64-encoded: `aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVszXTsKeCBxWzNdOwpjaCBxWzNdLCBxWzJdOwpjeCBxWzJdLCBxWzBdOwp4IHFbM107Cg==`

F# minor:

```qasm
include "qelib1.inc";
qreg q[4];

h q[0];
x q[0];
cx q[0], q[2];
cx q[0], q[1];
x q[0];
ch q[0], q[3];
```

Base64-encoded: `aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVswXTsKeCBxWzBdOwpjeCBxWzBdLCBxWzJdOwpjeCBxWzBdLCBxWzFdOwp4IHFbMF07CmNoIHFbMF0sIHFbM107`

G minor:

```qasm
include "qelib1.inc";
qreg q[4];

x q[1];
h q[2];
cx q[2], q[0];
x q[2];
ch q[2], q[3];
x q[2];
```

Base64-encoded: `aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCnggcVsxXTsKaCBxWzJdOwpjeCBxWzJdLCBxWzBdOwp4IHFbMl07CmNoIHFbMl0sIHFbM107CnggcVsyXTs=`

G# minor:

```qasm
include "qelib1.inc";
qreg q[4];

h q[3];
ch q[3], q[1];
x q[3];
cx q[3], q[1];
x q[3];
cx q[1], q[0];
```

Base64-encoded: `aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVszXTsKY2ggcVszXSwgcVsxXTsKeCBxWzNdOwpjeCBxWzNdLCBxWzFdOwp4IHFbM107CmN4IHFbMV0sIHFbMF07`

A minor:

```qasm
include "qelib1.inc";
qreg q[4];

h q[3];
cx q[3], q[0];
x q[3];
ch q[3], q[2];
x q[3];
```

Base64-encoded: `aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVszXTsKY3ggcVszXSwgcVswXTsKeCBxWzNdOwpjaCBxWzNdLCBxWzJdOwp4IHFbM107`

A# minor:

```qasm
include "qelib1.inc";
qreg q[4];

h q[0];
ch q[0], q[2];
x q[0];
cx q[0], q[1];
cx q[0], q[3];
x q[0];
```

Base64-encoded: `aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVswXTsKY2ggcVswXSwgcVsyXTsKeCBxWzBdOwpjeCBxWzBdLCBxWzFdOwpjeCBxWzBdLCBxWzNdOwp4IHFbMF07`

B minor:

```qasm
include "qelib1.inc";
qreg q[4];

x q[1];
h q[0];
cx q[0], q[3];
x q[0];
ch q[0], q[2];
x q[0];
```

Base64-encoded: `aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCnggcVsxXTsKaCBxWzBdOwpjeCBxWzBdLCBxWzNdOwp4IHFbMF07CmNoIHFbMF0sIHFbMl07CnggcVswXTs=`


Now that we know how to give the program specific notes, we can start interacting with it!

Using the netcat listener we were given, we can try C major a couple times and see what it gives us!

![Quantum Symphony Image 1](images/quantum1.png)

Turns out our first 4 chords are C major!

We can use Python and pwntools to automate the previously found chords in the progression to make figuring the rest out a bit easier!

```py
from pwn import *

r = remote('quantum-symphony.ctf.maplebacon.org', 1337)

prog = ["Cmaj", "Cmaj", "Cmaj", "Cmaj"]

CHORDS = {
"Cmaj": "aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVsyXTsKaCBxWzBdOwpjeCBxWzBdLCBxWzFdOwpjaCBxWzBdLCBxWzJdOwpjeCBxWzBdLCBxWzJdOw==",
"Dmaj": "aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVsxXTsKeCBxWzFdOwpjeCBxWzFdLCBxWzBdOwpjeCBxWzFdLCBxWzNdOwp4IHFbMV07CmNoIHFbMV0sIHFbMl07",
"D#maj": "aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCnggcVsxXTsKaCBxWzBdOwpjaCBxWzBdLCBxWzJdOwp4IHFbMF07CmN4IHFbMF0sIHFbM107CnggcVswXTs=",
"Emaj": "aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVszXTsKY2ggcVszXSwgcVswXTsKY3ggcVswXSwgcVsxXTsKeCBxWzNdOwpjeCBxWzNdLCBxWzJdOwp4IHFbM107",
"Fmaj": "aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVszXTsKeCBxWzNdOwpjaCBxWzNdLCBxWzJdOwp4IHFbM107CmN4IHFbM10sIHFbMF07CmN4IHFbMl0sIHFbMF07",
"Gmaj": "aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCnggcVsxXTsKaCBxWzNdOwp4IHFbM107CmNoIHFbM10sIHFbMl07CnggcVszXTsKY3ggcVszXSwgcVswXTsKY3ggcVsyXSwgcVswXTs=",
"G#maj": "aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVszXTsKeCBxWzNdOwpjaCBxWzNdLCBxWzBdOwp4IHFbM107CmN4IHFbMF0sIHFbMV07",
"Amaj": "aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVsyXTsKeCBxWzJdOwpjeCBxWzJdLCBxWzBdOwpjaCBxWzJdLCBxWzNdOwp4IHFbMl07Cg==",
"A#maj": "aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVsyXTsKY3ggcVsyXSwgcVswXTsKeCBxWzJdOwpjeCBxWzJdLCBxWzFdOwpjaCBxWzJdLCBxWzNdOwp4IHFbMl07Cg==",
"Cmin":"aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVswXTsKY3ggcVswXSwgcVsxXTsKY2ggcVswXSwgcVsyXTs=",
"C#min":"aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVszXTsKeCBxWzNdOwpjaCBxWzNdLCBxWzJdOwp4IHFbMl07CmNjeCBxWzNdLCBxWzJdLCBxWzBdOwp4IHFbMl07CnggcVszXTs=",
"Dmin":"aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVszXTsKY3ggcVszXSwgcVswXTsKeCBxWzNdOwpjaCBxWzNdLCBxWzJdOwpjeCBxWzJdLCBxWzBdOwp4IHFbMl07CmNjeCBxWzNdLCBxWzJdLCBxWzFdOwp4IHFbMl07CnggcVszXTs=",
"D#min":"aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCnggcVsxXTsKaCBxWzNdOwp4IHFbM107CmNoIHFbM10sIHFbMl07CnggcVsyXTsKY2N4IHFbM10sIHFbMl0sIHFbMF07CnggcVsyXTsKeCBxWzNdOw==",
"Emin":"aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVszXTsKeCBxWzNdOwpjeCBxWzNdLCBxWzJdOwp4IHFbM107CmNoIHFbMl0sIHFbMV07CmN4IHFbM10sIHFbMV07CmN4IHFbMV0sIHFbMF07Cg==",
"Fmin":"aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVszXTsKeCBxWzNdOwpjaCBxWzNdLCBxWzJdOwpjeCBxWzJdLCBxWzBdOwp4IHFbM107Cg==",
"F#min":"aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVswXTsKeCBxWzBdOwpjeCBxWzBdLCBxWzJdOwpjeCBxWzBdLCBxWzFdOwp4IHFbMF07CmNoIHFbMF0sIHFbM107",
"Gmin":"aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCnggcVsxXTsKaCBxWzJdOwpjeCBxWzJdLCBxWzBdOwp4IHFbMl07CmNoIHFbMl0sIHFbM107CnggcVsyXTs=",
"G#min":"aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVszXTsKY2ggcVszXSwgcVsxXTsKeCBxWzNdOwpjeCBxWzNdLCBxWzFdOwp4IHFbM107CmN4IHFbMV0sIHFbMF07",
"Amin":"aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVszXTsKY3ggcVszXSwgcVswXTsKeCBxWzNdOwpjaCBxWzNdLCBxWzJdOwp4IHFbM107",
"A#min":"aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCmggcVswXTsKY2ggcVswXSwgcVsyXTsKeCBxWzBdOwpjeCBxWzBdLCBxWzFdOwpjeCBxWzBdLCBxWzNdOwp4IHFbMF07",
"Bmin":"aW5jbHVkZSAicWVsaWIxLmluYyI7CnFyZWcgcVs0XTsKCnggcVsxXTsKaCBxWzBdOwpjeCBxWzBdLCBxWzNdOwp4IHFbMF07CmNoIHFbMF0sIHFbMl07CnggcVswXTs="
}

for chord in prog:
	c = CHORDS[chord]
	print(r.recv().decode())
	print(f"Sending... {chord}")
	r.send(c.encode()+b"\n")

for chord in CHORDS:
	print(f"{chord}: {CHORDS[chord]}")

r.interactive()
```

This code creates a connection with the server and repeatedly first recieves data from the server and sends it the next chord's base64-encoded QASM code. The chord progression is changed by just modifying the `prog` list!

Once the progression is done, it prints all the chords for easy access and then makes the session interactive so you can continue!

And for predicting the chord, we don't need to go through everything... It appears the music is in the key of C, which means we can most likely expect F major, G major, A minor, D minor, or of course, C major!

Trying this out, we can start getting more information, including that the next two chords are F major!

![Quantum Symphony Image 2](images/quantum2.png)

Doing this repeatedly, eventually, we get the progression `["Cmaj", "Cmaj", "Cmaj", "Cmaj", "Fmaj", "Fmaj", "Cmaj", "Cmaj", "Gmaj", "Fmaj", "Cmaj", "Gmaj"]`, which can give us the flag!

![Quantum Symphony Image 3](images/quantum3.png)

And our flag is `maple{qu4ntum_p34c3_c0d3_1s_m3m3nt0_m0r1}`!


# Rev: JaVieScript

**Points:** 100

**Author:** vie.pls

**Description:** Something something JS something

**Files:** [checker.js](files/javiescript/checker[1].js)

## Writeup

This was a really fun challenge and it was really interesting that JS was used in a reversing challenge!

Anyways, this challenge deals heavily with JavaScript's weird handling of type coercions...

This was the JavaScript code given:

```js
var flag = "maple{";
var honk = {};

async function hash(string) {
	const utf8 = new TextEncoder().encode(string);
	const hashBuffer = await crypto.subtle.digest('SHA-256', utf8);
	const hashArray = Array.from(new Uint8Array(hashBuffer));
	const hashHex = hashArray
	  .map((bytes) => bytes.toString(16).padStart(2, '0'))
	  .join('');
	return hashHex;
  }

async function constructflag() {
	const urlParams = new URLSearchParams(window.location.search);
	var fleg = "maple{";
	for (const pair of urlParams.entries()) {
		honk[pair[0]] = JSON.parse(pair[1]); 
	}

	if (honk.toString() === {}.toString()) {
		fleg += honk.toString()[9];
	}

	if (Object.keys(honk).length === 0) {
		const test = eval(honk.one);
		if (typeof test === 'number' && test * test + '' == test + '' && !/\d/.test(test)) {
			fleg += 'a' + test.toString()[0];
		}

		const quack = honk.two;

		if (quack.toString().length === 0 & quack.length === 1) {
			fleg += 'a' + (quack[0] + '')[0].repeat(4) + 'as';
		}

		const hiss = honk.three;

		if (hiss === "_are_a_mId_FruiT}") {
			fleg += hiss;
		}
	}
	if (await hash(fleg) == "bfe06d1e92942a0eca51881a879a0a9aef3fe75acaece04877eb0a26ceb8710d") {
		console.log(fleg);
	}
}

constructflag();
```

For the first part of our flag, it clearly begins with `maple{`, which is hardcoded in!

Then, we start getting into the reversing...

```js
	if (honk.toString() === {}.toString()) {
		fleg += honk.toString()[9];
	}
```

Here, it seems obvious that `honk` is an object. However, the `toString()` method of objects in JavaScript returns `[object Object]` rather than anything useful, and the 9th index of that is "b", so now we have `maple{b` as our flag!

The code checks that `honk` doesn't have any keys, but this isn't really useful to us... Nothing else is done to the flag outside of this, and the flag needs to be closed!

```js
		if (typeof test === 'number' && test * test + '' == test + '' && !/\d/.test(test)) {
			fleg += 'a' + test.toString()[0];
		}
```

Here, we see that `test` is a number by type and when multiplied by itself returns itself, but also does not match some regex for digits. Well, the one value that satisfies all of these in JavaScript is `NaN`! It can be a huge pain when it gives you that from dividing by 0 without telling you in a warning/error, but here, it is useful, since it gives us the next letters of our flag, which is now `maple{baN`!

```js
		if (quack.toString().length === 0 & quack.length === 1) {
			fleg += 'a' + (quack[0] + '')[0].repeat(4) + 'as';
		}
```

Here, we find that `quack` is somehow something with a length that returns an empty string. The only things in JavaScript with a length are strings and arrays, so it must be an array! It also has one thing inside it. The only two possible values here are `[undefined]` or `[null]` and `maple{baNauuuuas` doesn't really make sense, so it must be the latter, giving us `maple{baNannnnas`!

```js
		if (hiss === "_are_a_mId_FruiT}") {
			fleg += hiss;
		}
```

And another easy part, just add this to our flag, and our final flag is `maple{baNannnnas_are_a_mId_FruiT}`!


# Misc: Maple Island <3

**Points:** 100

**Author:** hiswui

**Description:** The name of the game is simple. It's love. They say opposites attract. You know like North and South, Hot and Cold, etc. The same is said to be true for parity too, the odd (the ones) and even DWORDS (the zeroes) have always had quite steamy and passionate relationships.

Historically speaking, tradition was paramount for this species. The zeroes scour the world in hopes of find their special One. (Where do you think the saying comes from? duh.) However, we are in the 21st century and must adapt to the new.

So, we made an entire reality TV show about it. The premise is simple: Screw tradition, in this show, only the Ones are allowed to court the zeroes.

Stay tuned for the most drama-filled season of Maple Island as of yet with even more tears, arguments, and passionate moments than ever before. Will every match made in Maple heaven be stable?

Maple Island streaming next month on MapleTV!

But wait, lucky viewers have a chance to catch exclusive early-access content if they can solve the following puzzle below and text the answer to 1-800-MAPLE-1337.

`nc maple-island.ctf.maplebacon.org 1337`

**Files:** [dist.zip](files/island/dist[1].zip)

## Writeup

Inside the zip file is a very useful Python script!

This is the code for that script:

```py
import os
import time
import random
from secrets import SECRET_FLAG_TEXT, create_a_perfect_world


CONTESTANTS_PER_SIDE = 20
def generate_contestants():
    ones = []
    zeroes = []
    oprefs = []
    zprefs = []
    while len(ones) < CONTESTANTS_PER_SIDE or len(zeroes) < CONTESTANTS_PER_SIDE:
        contestant = os.urandom(4);
        if contestant[-1] % 2 == 0 and len(zeroes) < CONTESTANTS_PER_SIDE:
            zeroes.append(contestant)
        elif contestant[-1] % 2 == 1 and len(ones) < CONTESTANTS_PER_SIDE:
            ones.append(contestant)

    for i in range(CONTESTANTS_PER_SIDE):
        oprefs.append(random.sample(zeroes, len(zeroes)))
        zprefs.append(random.sample(ones, len(ones)))
    return (ones, zeroes, oprefs, zprefs)



def xor_streams(otp, flag):
    assert len(otp) == len(flag)
    return b"".join([int.to_bytes(x ^ y) for x, y in zip(otp, flag)])




print("""

WELCOME TO MAPLE ISLAND <3
---------------------------------

The name of the game is simple. It's love. They say opposites attract.
You know like North and South, Hot and Cold, etc. The same is said to
be true for parity too, the odd (the ones) and even DWORDS (the zeroes) 
have always had quite steamy and passionate relationships. 
      
Historically speaking, tradition was paramount for this species. The
zeroes scour the world in hopes of find their special One. (Where do
you think the saying comes from? duh.) However, we are in the 21st 
century and must adapt to the new.

So, we made an entire reality TV show about it. The premise is simple:
Screw tradition, in this show, only the Ones are allowed to court the 
zeroes.
 
Stay tuned for the most drama-filled season of Maple Island as of yet
with even more tears, arguments, and passionate moments than ever before.
Will every match made in Maple heaven be stable?
      
Maple Island streaming next month on MapleTV!
      
But wait, lucky viewers have a chance to catch exclusive early-access content
if they can solve the following puzzle below and text the answer to 1-800-MAPLE-1337.
""")

# just for readability on terminal
time.sleep(2)

ones, zeroes, oprefs, zprefs = generate_contestants()
otp = b''
for couple in create_a_perfect_world(ones, zeroes, oprefs, zprefs):
    otp += couple



ctext = xor_streams(otp, SECRET_FLAG_TEXT)
print(f"""
ones: {ones}
zeroes: {zeroes}
oprefs: {oprefs}
zprefs: {zprefs}
ctext: {ctext}
""")
```

This doesn't really tell us much other than that the flag is XOR-encoded with a key generated from couples of "ones" and "zeroes" and that each "one" is an odd number converted to bytes and every "zero" is an even number.

Anyways, we can go ahead and connect to get our data!

![Maple Island Image 1](images/island1.png)

The easiest thing to do here is convert the bytes in ctext to hexadecimal. Opening a Python shell and using the `hex()` method for the bytes object works perfectly well for this!

Anyways, since we're dealing with XOR, this is a great job for [CyberChef](https://gchq.github.io/CyberChef/)!

This is a lot of text, so it's probably not all the flag, so we can try looking to see if the end is a `}`! It might not be, but it's worth looking! We can put a bunch of spaces with this character at the end as an XOR key and see what that last byte is!

![Maple Island Image 2](images/island2.png)

And our last byte is `ca`! This is even, making it most likely a zero, and we can parse through the ones and zeroes a bit with Python to get the list and compare!

![Maple Island Image 3](images/island3.png)

And the very first item here is `244d82ca` which matches! There is another match, but this is only important if this first one doesn't work!

And copying the result and pasting it as the key in hex, modifying the bytes to include this "zero" at the end, gives us something that looks like it very well could be part of a key!

![Maple Island Image 4](images/island4.png)

Now, if we put the last "one" (`4591a73b`), assuming they are in order based on the ones courting the zeroes from the description, right before this, we get a bit more!

![Maple Island Image 5](images/island5.png)

It looks like the key is made up of the ones followed by the zeroes, so if we put all of the ones with `00000000` as the corresponding zero, we can get a bit that is useful!

![Maple Island Image 6](images/island6.png)

Half of this is broken up, but it seems to say "Love will never give up on you. Love will never let you down. Love will never run around and desert you," followed by the flag! If we try using the first item of `oprefs` as the zero instead of just `00000000`, we can confirm this and get a bit more!

![Maple Island Image 7](images/island7.png)

Again, we don't have everything, but we still have a lot! Now, we can just go around the messed up areas and replace those zeroes with other random zeroes from the list until we get printable text!

![Maple Island Image 8](images/island8.png)

And when we do that, we get the flag, `maple{G05h_1_w4nt_4_st4bl3_m4tch_t00_pls_1_4m_50_l0n3ly}`!


# Crypto: Pen and Paper

**Points:** 100

**Author:** vEvergarden

**Description:** A sweet classical cipher. Why use a computer when I have a pen and paper?

NOTE: please wrap the flag in maple{...}

**Files:** [ciphertext.txt](files/vigenere/ciphertext[1].txt) [source.py](files/vigenere/source[1].py)

## Writeup

First step as always is to look at the code! Ultimately it is a variant of a classical cipher, but the modifications make a huge difference!

```py
import string
import random

ALPHABET = string.ascii_uppercase


def generate_key():
    return [random.randint(0, 26) for _ in range(13)]


def generate_keystream(key, length):
    keystream = []
    while len(keystream) < length:
        keystream.extend(key)
        key = key[1:] + key[:1]
    return keystream


def encrypt(message, key):
    indices = [ALPHABET.index(c) if c in ALPHABET else c for c in message.upper()]
    keystream = generate_keystream(key, len(message))
    encrypted = []

    for i in range(len(indices)):
        if isinstance(indices[i], int):
            encrypted.append(ALPHABET[(keystream[i] + indices[i]) % 26])
        else:
            encrypted.append(indices[i])

    return "".join(encrypted)


with open("plaintext.txt", "r") as f:
    plaintext = f.read()

key = generate_key()
ciphertext = encrypt(plaintext, key)

with open("ciphertext.txt", "w") as f:
    f.write(ciphertext)
```

This gives us a lot of information! First, we know the key is 13 numbers long and each number is between 0 and 26 (including 0)! Second, we can see that this is very similar to a running-key Vigenere cipher. And lastly, we know how the key stream is generated, simply moving the first number to the end each iteration!

And our first step is to use this to generate our own keystream! If we use a key of 1234567890abc, we can see what characters match up with what key, which can help during cryptanalysis!

![Pen and Paper Image 1](images/vigenere1.png)

Now, we need to generate a readable way to view this alongside the ciphertext! Setting `code` to the ciphertext and `key` to our new keystream, we can run the following JavaScript script in the console to get this format!

![Pen and Paper Image 2](images/vigenere2.png)

And just pasting that into a text file, we can start decoding!

![Pen and Paper Image 3](images/vigenere3.png)

First, we can see a bunch of single-letter words... Luckily, this can only be one of two things: "a" or "I". And it is most likely "a" since a majority of things other than personal letters won't use "I" as much...

Specifically, we find the following instances:

* An M with a 0 corresponding to it in the key
* A T with a 7 corresponding to it in the key
* A P with a 9 corresponding to it in the key
* Another T but now with a 4 corresponding to it in the key
* An I with a 5 corresponding to it in the key
* A V with a 2 corresponding to it in the key

For each of these, we can find a part of the key as follows, with the example being the first:

```py
(ord('M') - ord('A'))%26
```

Just changing M to the ciphertext letter and A to the plaintext letter, this is easy to do!

Now, considering how the key values in our stream are one greater than the index (assuming 0 as 10, a as 11, b as 12, and c as 13), we can get a few indices of the key!

Index 9 will be 12.

Both index 6 and index 3 will be 19.

Index 8 will be 15.

Index 4 will be 8.

Index 1 will be 21.

We can add the following to `source.py` to control these indices and make the rest 0 so we are left with the original ciphertext:

```py
    k[0] = 0
    k[1] = 21
    k[2] = 0
    k[3] = 19
    k[4] = 8
    k[5] = 0
    k[6] = 19
    k[7] = 0
    k[8] = 15
    k[9] = 12
    k[10] = 0
    k[11] = 0
    k[12] = 0
```

And running, we can decrypt some of the cipher!

![Pen and Paper Image 4](images/vigenere4.png)

Two obvious changes in the first couple words are that "AVD" should be "AND" and "BO" should be "TO", so we can end up finding that both index 2 and 5 of the key are 8!

And decrypting a bit more with this!

![Pen and Paper Image 5](images/vigenere5.png)

Now, it's starting to look like the beginning says "COMPETITIONS AND CRYPTOGRAPHY", so using this we can get index 0 as 12, index 7 as 23, index 10 as 25, index 11 as 16, and index 12 as 12! And this completes our key, so we can decrypt and get our flag!

![Pen and Paper Image 6](images/vigenere6.png)

And our flag is `maple{VIGENEREWITHAWTIST}`!
