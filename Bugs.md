# uGrind bugs


We studyμgrindby injecting bugs into an existing open-source HLS framework. These injected bugs are indicativeof  the  style  of  bugs  that  a  designer  might  encounter  whenusing HLS. In this study, we use the Relu program with  the  rather  large  input  vector  size  of N=256.We  capture  these  bugs  on  Verilog  simulation  and  AWS  F1instances, Xilinx UltraScale+ FPGA, with the on-chip SRAMlimit of 33MB. We evaluate the amount of on-chip memoryrequired by theguards and demonstrate that they fit within theFPGA constraints. We also study the number of verificationiterations required to pinpoint the bug. This is a function of thememory resources on the board. We explore using a smaller FPGA board, Cyclone V FPGAs with the 1.2MB. In this case,the number of iterations will increase as we can only partially guard the circuit.


## Bug1  Defective  Adders

Typically,  HLS  includes  RTLblack  boxes  for  primitive  operations  such  as  adders.  Whilethe  individual  black  boxes  may  be  verified,  HLS  couldunintentionally introduce errors in their operation by not settingall the required signals. In this case study, all the adders willbehave  faulty,  causing  multiple  bugs  that  overlap  with  eachother. The misbehaviour of these adders will affect the outputsof  branch  and  store,  which  are  the  nodes  we  guard  in  theverifier’s first iteration. This requires the verifier to guard allthese nodes. We plot the memory consumed by the guards ineach iteration of the verifier in Figure 8. In the second iteration,the memory usage hits the peak as we track the backward sliceof the branch and store (which failed verification in the firstiteration). This implies guarding every node in the innermostloop. In the third iteration, the backward slice reaches the addnodes,  which  are  promptly  detected  to  be  the  bug  sites.  In the Figure  below,   we  plot  what  happens  when  we  verify  using  thesmaller Cyclone V board with limited on-chip memory. Thelimited memory narrows the scope of the backward slice thatcan be guarded, triggering more verification iterations.

![alt text](https://i.ibb.co/pytmNwv/paper.png "Figure 1")


## Bug2 Bug  2:  Incorrect  memory  address

Parameterized HLS compilers generate buses that supportdifferent address width. In this case, a possible bug is data bit-width mismatch between the accelerator and the bus, leadingto corruption in values load and stored. We consider the casethat an accelerator with the data bit-width of 32 is connected toa host processor with the bit-width of 64, causing the memory addresses  to  be  miscalculated.  In  this  case,  the  gep  node generates memory addresses for both load and store. We startthe verification procedure by guarding the store and live outof the circuit. As we proceed backwards through the store’sbackward  slice,  we  will  pin  down  the  fault  on  the  address calculation. Figure 8 plots the number of iterations required and memory usage, assuming 33MB is available for the guards.Figure 9 plots the number of iterations required (if a smallerFPGA)
