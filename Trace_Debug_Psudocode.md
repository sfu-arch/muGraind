
# Comparing the process of catching multiple bugs in a tarace based approach VS uGrind


## uGrind

The process is entirely automatic, the user will set the inputs and select the C program, the automated iterative verifier will start verifying the circuit in 3 or more iterations. This process might take minutes or hours based on the user's decision to simulate or synthesis. At the end the user will be presented with a list of all the buggy locations in the circuit, they may then start fixing the bugs.

uGrind iterative verifier flow:

User sets program and input
  User chooses to generate the circuit with uGrind enabled
    uGrind iterative verifier will start vrifiying the circuit
User is presented with a list of places of mismatch

## Trace-Based

Here, the user should first set the program and input as our case. After the hardware execution a trace will be dumped. This large trace needs to be then post-processed to find the first place of mismatch. This first place of mismatch will only mark one of the bugs. The user then has to debug this exact bug and re run the circuit and go through the whole process again to locate the second bug. 

Trace based verifier flow:


For (all bugs to be found: no more mismatches){
  User sets program and input
      User chooses to generate the circuit with trace enabled
        Circuit runs and Trace is dumped (Large, memory overhead)
          Trace is post-processed to find the first place of mismatch (Time consuming, Trace is large)
            The user finds the first place of mismatch and attempts to fix the bug (if the attempt is succesful the loop moves to the next mismatch, if not, stuck on the same place)
}

The whole process is very time consuming, and needs user to be present in the loop. Finding all the bugs depends on the users ability to fix the ones that showed up first


            
            



