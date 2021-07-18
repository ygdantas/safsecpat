-----------------------------------------------------------------
SafSecPat
-----------------------------------------------------------------
SafSecPat is a machinery for enabling automated reasoning with safety and security patterns, as well as consequences of using such patterns.

SafSecPat has been presented in the following article (under submission):

Yuri Gil Dantas, and Vivek Nigam. Automating Safety and Security Co-Design through Semantically-Rich Architectural Patterns, TCPS 2021.

-----------------------------------------------------------------
Purpose of this README file
-----------------------------------------------------------------
This README file provides instructions on how to reproduce the results presented in the article. 

-----------------------------------------------------------------
Setup
-----------------------------------------------------------------
The instructions have been tested in a Linux operating system (Ubuntu 18.04 desktop 64-bit).

-----------------------------------------------------------------
Files
-----------------------------------------------------------------
- safety/safety.dlv contains the safety reasoning principle rules
- security/security.dlv contains the security reasoning principle rules
- tradeoff/tradeoff.dlv contains the reasoning principle rules for safety and security consequences
- utils/utils.dlv contains auxiliary functions 
- examples/headlamp/ contains the architecture of the headlamp system

-----------------------------------------------------------------
Instruction for running the Headlamp system
-----------------------------------------------------------------
(0) Run the following command to explore which safety patterns are recommended to tolerate the identified failures

./dlv examples/headlamp/safety-architecture-headlamp.dlv utils/utils.dlv safety/safety.dlv -nofinitecheck -filter=safetyPattern -filter=satisfiedSG -filter=avoided

The command (0) will provide 3 complete solutions that satisfied the defined safety goals tolerate all identified failures. We present the following solution in the paper:

{safetyPattern([nuIDSAFPAT,monitorActuator,1],[monitorActuator,[cam,[nuREDUNDANCY,1],[nuCHECKER,1]],[nuINPUTS,1],[nuINTERNAL,1],[nuOUTPUTS,1]]), safetyPattern([nuIDSAFPAT,dualSelfCheckingPairFS,1],[dualSelfCheckingPairFS,[bdCtl,[nuREDUNDANCY,1],[nuCHECKER,1]],[nuINPUTS,1],[nuINTERNAL,1],[nuOUTPUTS,1]]), avoidedby(idfl3,cam,[[nuIDSAFPAT,monitorActuator,1],b,never,most1fail,never]), avoidedby(idfl1,bdCtl,[[nuIDSAFPAT,dualSelfCheckingPairFS,1],d,all1fail,never,all2fail]), satisfiedSG(sg1), satisfiedSG(sg2)}

Note that we set the following constraint (in safety-architecture-headlamp.dlv) to only show solution where all failures have been avoided

:- not satisfiedSG(IDSG), sg(IDSG,_) .

(1) Run the following command to explore which security patterns are recommended to mitigate the identified threats

./dlv examples/headlamp/security-architecture-headlamp.dlv utils/utils.dlv security/security.dlv -nofinitecheck -filter=securityPattern -filter=mitby

The command (1) will provide 2 complete solutions that mitigate all identified threats. We present the following solution in the paper: 

{securityPattern([nuIDSECPAT,firewall,1],[firewall,[can2,ecu3,[nuCHECKER,1]],[nuINPUTS,1],[nuINTERNAL,1],[nuINTERNAL,1]]), mitby([idfl1,[ecu2,can2,ecu3,can1,int2]],bdCtl,[[nuIDSECPAT,firewall,1],firewall]), mitby([idfl1,[ecu2,can2,ecu3,can1,int3]],bdCtl,[[nuIDSECPAT,firewall,1],firewall]), mitby([idfl1,[ecu2,can2,ecu3,can3,int1]],bdCtl,[[nuIDSECPAT,firewall,1],firewall]), mitby([idfl2,[can2,ecu3,can3,int1]],bcps,[[nuIDSECPAT,firewall,1],firewall]), mitby([idfl2,[can2,ecu3,can1,int3]],bcps,[[nuIDSECPAT,firewall,1],firewall]), mitby([idfl2,[can2,ecu3,can1,int2]],bcps,[[nuIDSECPAT,firewall,1],firewall])}


Note that we set the following constraint (in security-architecture-headlamp.dlv) to only show solution where all threats have been mitigated

:- nmit(X).

(2) Run the following command to show the safety consequences of deploying security patterns

./dlv examples/headlamp/security-architecture-headlamp.dlv utils/utils.dlv security/security.dlv tradeoff/tradeoff.dlv -nofinitecheck -filter=ft -filter=fl -filter=ft2fl

The command (2) will show the new faults and failures caused by the deployment of security patterns.

(3) Run the following command to show the security consequences of deploying safety patterns

./dlv examples/headlamp/safety-architecture-headlamp.dlv utils/utils.dlv safety/safety.dlv -nofinitecheck tradeoff/tradeoff.dlv -filter=pThreat

The command (3) will show the new potential threats caused by the deployment of safety patterns.

