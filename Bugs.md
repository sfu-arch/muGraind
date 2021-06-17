# uGrind bugs


We studyμgrindby injecting bugs into an existing open-source HLS framework. These injected bugs are indicativeof  the  style  of  bugs  that  a  designer  might  encounter  whenusing HLS. In this study, we use the Relu program with  the  rather  large  input  vector  size  of N=256.We  capture  these  bugs  on  Verilog  simulation  and  AWS  F1instances, Xilinx UltraScale+ FPGA, with the on-chip SRAMlimit of 33MB. We evaluate the amount of on-chip memoryrequired by theguards and demonstrate that they fit within theFPGA constraints. We also study the number of verificationiterations required to pinpoint the bug. This is a function of thememory resources on the board. We explore using a smaller FPGA board, Cyclone V FPGAs with the 1.2MB. In this case,the number of iterations will increase as we can only partially guard the circuit.

The Relu circuit 
![alt text](https://i.ibb.co/kD6xFGH/paper2.png)


## Case 1  Defective  Adders

Typically,  HLS  includes  RTLblack  boxes  for  primitive  operations  such  as  adders.  Whilethe  individual  black  boxes  may  be  verified,  HLS  couldunintentionally introduce errors in their operation by not settingall the required signals. In this case study, all the adders willbehave  faulty,  causing  multiple  bugs  that  overlap  with  eachother. The misbehaviour of these adders will affect the outputsof  branch  and  store,  which  are  the  nodes  we  guard  in  theverifier’s first iteration. This requires the verifier to guard allthese nodes. We plot the memory consumed by the guards ineach iteration of the verifier in Figure 8. In the second iteration,the memory usage hits the peak as we track the backward sliceof the branch and store (which failed verification in the firstiteration). This implies guarding every node in the innermostloop. In the third iteration, the backward slice reaches the addnodes,  which  are  promptly  detected  to  be  the  bug  sites.  In the Figure  below,   we  plot  what  happens  when  we  verify  using  thesmaller Cyclone V board with limited on-chip memory. Thelimited memory narrows the scope of the backward slice thatcan be guarded, triggering more verification iterations.

![alt text](https://i.ibb.co/pytmNwv/paper.png "Figure 1")


## Case 2 Incorrect  memory  address

Parameterized HLS compilers generate buses that supportdifferent address width. In this case, a possible bug is data bit-width mismatch between the accelerator and the bus, leadingto corruption in values load and stored. We consider the casethat an accelerator with the data bit-width of 32 is connected toa host processor with the bit-width of 64, causing the memory addresses  to  be  miscalculated.  In  this  case,  the  gep  node generates memory addresses for both load and store. We startthe verification procedure by guarding the store and live outof the circuit. As we proceed backwards through the store’sbackward  slice,  we  will  pin  down  the  fault  on  the  address calculation. Figure below plots the number of iterations required and memory usage, assuming 33MB is available for the guards. the number of iterations required if a smaller FPGA was at hand was prlotted previosly. 

![alt text](https://i.ibb.co/cTRGR1J/paper1.png)




## Case 3 Mixed-up Signals 
A possible bug that can happenduring H-RTL generation is the mixing up of control signals.This akin to mixing up the inputs to a mux. For instance, theorder of connections to select nodes matches the bitmask inthe input of the select signal. In this case, while the inputs tothe select node are correct, the select node’s output is wrongbecause the select node has selected a wrong operand. In the Relu test case, we injected a bug that would affect select5 inevery  iteration  on  both  outputs,  showing  it  affects  both  thestore and branch node.


## Case 4 Incorrect Handshaking
H-RTL correctness implic-itly depends on handshaking between the nodes. In this bug,we consider a case where a node prematurely fires a dependentoperation before the output has stabilized, causing the consumerto latch an incorrect value. We introduce fault in the multiplier(mul3) in the outer loop, which causes the adder6 and live-outs to misbehave. Since mul3 is a live-out, it will be guarded onthe first iteration, and the mismatch is identified early. Sincemost of the H-RTL circuit is included in the unguarded innerloop, we require minimal memory to guard the buggy node.

# Real life bug in open-source HLS uIR


n this case study, we walk through the process of identifyinga bug in existing HLS software. The bug was found in commitSHA: #f ae f daof theμIRHLS compiler (https://github.com/sfu-arch/muir). We walk through the steps of how μgrind made decisions about when to guard and what to guard, which ledto the successful identification of the defective module. This particular bug led to incorrect behavior on two applications,Stencil2D and FFT from Machsuite. In our experiment, tokeep debugging time tractable we focused on Stencil and setthe input size for the stencil to64×128data points, the filtersize is3×3, and the data bounds were betweenMAX=1000andMIN=1.First,  we  ran  the  H-RTL  of  the  Stencil2D  and  comparedthe memory state against the software. Figure below b shows the differences between data values. We found that the output data matched for the first three data points, but it following points were all incorrect.In  the  first  step,μgrind tries  to  isolate  the  buggy  part  ofthe  design  by  guarding  the  live-ins,  live-outs  and  memory operations.μgrind filled  the guard shadow  RAMs  with information  from  the  software  trace  to  analyze  the  nodesthat mismatched. 
10(c) shows the memory state after running the debug version of Stencil2D. The guards patch in thecorrect values from the software and only one in four memory locations  displays  incorrect  values.  Furthermore,  the  guards marked up the particular H-RTL node that read or wrote those locations. Following this, we instrumented and guarded onlythe flagged H-RTL node, and turned on heavyweight guardingthat  analyzed  all  the  incoming  and  outgoing  ports  from  the node. This included theread line and write ports to and from memory. In the post-execution processing, we identified We can see that the request address to the memory is correctin the post-processing analysis phase, but the cache’s return value is wrong. Hence,μgrind helps the user flag the cache asthe source of the bug. After this step, we usedguards into theproblem  and  look  at  the  memory  traces  of  the  cache.  Whatwe observed from the traces is that whenever there is a cache miss, after every four accesses, the returned value for the 5thelement  is  wrong  and  the  data  is  the  same  as  the  previousactive write request, store node with the ID of 40, to the cache.From this observation, we conclude that the bug in the cacheshows up whenever there is a read request to the cache, whichis not a hit, and there is an active write request sent just before the load request.To summarize, while the traditional HLS debugging tech-niques can not be used to uncover errors in interfaces betweenblocks, when some of the blocks are not developed using anHLS-based methodology, using μgrind could successfully helpus find the location and reason for an existing bug (that is notauto-generated by the compiler) in μIR hardware modules.

![alt text](https://i.ibb.co/12th2hF/paper3.png)

