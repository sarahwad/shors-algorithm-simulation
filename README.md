# shors-algorithm-simulation

# Design and implementation of the Ufa,N circuit of Shor Algorithm. 

## Understanding the Algorithm:
Shor’s algorithm is the problem factoring integers in polynomial time O(n^2 log n log log n) by reducing the factoring problem to a period finding problem. In this report, we’ll call the number we want to factor N. We need to choose an integer a randomly such that: 1 < a < N and does not have a nontrivial factor with N. This can be done with Euclidian algorithm to calculate GCD(a, N), if result is not 1 it’s not coprime and we need to pick another value for a.

To find the period r, We need to find powers of a (mod N). We need to solve for:
f(a,N)(x) = a^x mod N

The period r has to be an even number to find factors. If we get an odd number, we need to do it again with a different value for a. 
After finding r, we find GCD((a^(r/2) +1), N) and GCD((a^(r/2) -1), N) to find prime factor, if we get a one nontrivial result we can test if it’s a factor of N by dividing it for N to get the other prime factor. Doing this for larger N values is not possible in classical computers. Which makes this step the quantum part of the algorithm. The circuit for shor algorithm is as such:

<img width="713" alt="Screen Shot 2021-07-10 at 10 00 18 AM" src="https://user-images.githubusercontent.com/63491500/125154988-a3252880-e165-11eb-80f9-b98dbe3c87d6.png">

We first need n, the binary representation of our N. We will need to evaluate fa,N for at least the first N^2 values of a and so will need at least 2n qubits in our circuit. The output of this function will always be less than N, and so we will need n = ⌈𝑙𝑜𝑔2 𝑁⌉ output bits.

The special part is ability to be in a superposition to calculate fa,N(x) for all needed a. So we need to initialize our qubits with Hadamard gate to put them superposition. Implementing the period finding function. Each successive application of U will multiply the state of our register by a(mod N) and after applications we will arrive at the state |1⟩ again. Superposition of the states in this cycle is an eigenstate of U which has an eigenvalue of 1. It can be represented as an eigenstate with the a different phase for each of the computational basis states which contain the period r that we want. Since computational basis state is a superposition of these phases eigenstates, we can use quantum phase Eestimation on U to measure a phase which has the period r as a denominator.


## Implementation:
The algorithm implemented using Qiskit with python Jupyter Notebook. Number of N is 55 and the picked value a = 8. As explain in the previous section, if N = 55, it can be represented n = ⌈𝑙𝑜𝑔2 55⌉ = 6 qubits. We will need 2n quantum qubits to evaluate our function at N2 times, the output qubits are n as it will always be less than value of N.

Before creating the circuit, we define functions for each step in Shor Algorithm described in Yanofsky book:
Step 1: Use a polynomial algorithm to determine if N is prime or a power of prime. If it is a prime, declare that it is and exit. If it is a power of a prime number, declare that it is and exit:
The picked N has to be a nontrivial number (N>1) and check if N is prime or a power of prime. To check value of N, here generalized version is used.
For easier use, N is selected manually as a user input.

 
Step 2: Randomly choose an integer a such that 1 < a < N. Perform Euclid’s algo- rithm to determine GCD(a, N). If the GCD is not 1, then return it and exit. The generalized code. Like previous step, a user input is used to pick a for easier implementation. This code consumes time for big values of N

Step 3: Use quantum circuit (6.166) to find a period r. The over all circuit will be implemented as follows:

<img width="942" alt="Screen Shot 2021-07-10 at 10 10 57 AM" src="https://user-images.githubusercontent.com/63491500/125155259-36129280-e167-11eb-82d2-316b670a4708.png">

 First, we begin by initializing the qubits. On the first 2n qubits, we apply a hadamard gate to put in superposition of all computational basis states and target qubit 1. After initializing our qubits, we map the modular exponentiation operations to the measurement qubits by applying the unitary operator U with various powers onto the target qubits while controlling it with each of the different measurement qubits. We begin by calling the multiplication function for each qubit by sending a and its power a^2i for each i as number of qubits in mod N.  
 
This function will only accepts a = 2,4,7,8,13,42,47,48,51,53 coprimes values with N. For each value of a the multiplication gate has to be unitary. For values a = 2 or 53 the unitary gate will look like below where a = 2 in 6-bit binary is 000010 can be seen represented before the first barrier of swaps where q_5 is our auxiliary qubit = 1. When the power is 2^1 (a_x_mod55 function for a^2^1) a will be repeated twice and the output will be 000100 = 4 with the same operator:

The same thing can be seen for other values of a: a = 4 or 51
a = 8 or 47
a = 7 or 48
a = 13 or 42
 
Then we expose the period by applying an inverse quantum Fourier transform on the n measurement qubits since the quantum phase estimation is encoded the modular exponentiation functions with a fourier basis. Applying an inverse QFT translates that fourier basis into the computational basis revealing periodic elements.
The resulted circuit:
     
Step 4: we measure the first n qubits.
After simulating our circuit we go over each measurement and find the phase by dividing by 2^i for all qubits i. taking the fraction we’ll get our period r as the denominator. After getting possible values of r, we filter by picking even r values. Below is the code with output:
The last step, we find factors based on r values we got by using GCD((a r/2 +1), N) and GCD((a r/2 -1), N) as mentioned above. After that we look for nontrivial factors from our result as consider it our prime factor p then test for q by diving N/p. below is detailed code:
    
 References:
[1] Yanofsky “Quantum Computing for Computer Scientist” (2008) [2] Qiskit: Shor's Algorithm
[3] IBM Quantum Experience: Shor’s Algorithm
 
