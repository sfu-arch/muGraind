# Defining the instrumented circuits

In the Scala files, each instruction node, which is a higher level definition of a hardware implementation of an instruction in the circuit, has a Debug input flag which is set to either true or false at the beginning. When a Debug flag for an instruction is set to 1 that instruction will be guarded during the hardware execution, meaning that the guard corresponding to that instruction's ID will be enabled and connected to that instruction using Bore connections.

Since enabling and disabling guards has to be implemented automatically and without user interference, we pass a Json file to the hardware generator that contains the properties of the circuit, including the number of instructions that are being guarded "NumDBGs" and the IDs of the instructions that are being guarded "boreIDs"
