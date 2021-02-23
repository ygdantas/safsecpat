-----------------------------------------------------------------
SafSecPat
-----------------------------------------------------------------
SafSecPat is a domain-specific language for enabling automated reasoning with safety and security patterns, as well as consequences of using such patterns.

SafSecPat has been introduced by the following article submitted to SAFECOMP 2021:

Yuri Gil Dantas, and Vivek Nigam: 
Automated Reasoning for Safety and Security Co-Engineering

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
- safety/safety.dlv: contains our safety reasoning with safety patterns 
- security/security.dlv: contains our security reasoning with security patterns 
- tradeoffs/tradeoffs.dlv: contains our reasoning rules on consequences of using patterns
- utils/utils.dlv: contains some utils rules 
- dlv: Linux version of dlv
- examples/headlamp/ : Headlamp System (HLS) case study
     - headlamp/architecture-headlamp-not-ctl-saf-1.dlv : contains architecture of HLS for safety exploration mode
     - headlamp/architecture-headlamp-not-mit-sec-1.dlv : contains architecture of HLS for security exploration mode
     - headlamp/architecture-headlamp-not-ctl-saf-2.dlv : contains architecture of HLS for co-analysis: consequences on safety
     - headlamp/architecture-headlamp-not-mit-sec-2.dlv : contains architecture of HLS for co-analysis: consequences on security
- examples/battery/ : Battery Management System (BMS) case study
     - headlamp/architecture-battery-not-ctl-saf-1.dlv : contains architecture of BMS for safety exploration mode
     - headlamp/architecture-battery-not-mit-sec-1.dlv : contains architecture of BMS for security exploration mode
     - headlamp/architecture-battery-not-ctl-saf-2.dlv : contains architecture of BMS for co-analysis: consequences on safety
     - headlamp/architecture-battery-not-mit-sec-2.dlv : contains architecture of BMS for co-analysis: consequences on security

-----------------------------------------------------------------
Instruction for running the HLS case study
-----------------------------------------------------------------
(0) Run the following command to explore which safety patterns are recommended to tolerate the identified failures

./dlv examples/headlamp/architecture-headlamp-not-ctl-saf-1.dlv safety/safety.dlv utils/utils.dlv -filter=tolby -filter=ntol -filter=safMon -filter=watchDog -filter=tmr

The command (0) will provide 3 complete solutions that tolerate all identified failures. We present solution b) in the paper.

a)
-> watchDog(nuWD,hdlmpSystem,nuscwd,nulvwd)
-> tmr(nuTMR,bodyCtrl,nucp2,nucp3,nuchm1,nuchm2,nuchm3,nuvtcp,nucho,nucpo)
-> safMon(nuSafMon,bodyCtrl,[allInputs],[allOutputs],nuSC,[numin],[numout])

b) 
-> watchDog(nuWD,hdlmpSystem,nuscwd,nulvwd)
-> safMon(nuSafMon,bodyCtrl,[allInputs],[allOutputs],nuSC,[numin],[numout])

c) 
-> watchDog(nuWD,hdlmpSystem,nuscwd,nulvwd)
-> tmr(nuTMR,bodyCtrl,nucp2,nucp3,nuchm1,nuchm2,nuchm3,nuvtcp,nucho,nucpo)


The above command will only show solutions (i.e., architectures) that all failures are tolerated by the suggested safety
patterns. To also show solutions where where not all failures are tolerated, make sure the following line is commented
out in examples/headlamp/architecture-headlamp-not-ctl-saf-1.dlv (needs manual change)

:- nctl([Z,V,P]).

(1) Run the following command to explore which security patterns are recommended to mitigate the identified threats

./dlv examples/headlamp/architecture-headlamp-not-mit-sec-1.dlv  security/security.dlv utils/utils.dlv -nofinitecheck -filter=firewall -filter=nmit -filter=secMonCP -filter=secMonCH -filter=mitby

The command (1) will provide 20 complete solutions that mitigate all identified threats. Three of these solutions are shown below. We present solution c) in the paper.

a)
-> firewall(nuFirewall,can2,gw,[allInputs],[allOutputs])
-> secMonCH(nuSecMonCH,obdGw,dec,min,mout)

b)
-> firewall(nuFirewall,can2,gw,[allInputs],[allOutputs])
-> secMonCP(nuSecMonCP,bodyCtrl,[ic],[oc],dec,[min],[mout])

c)
-> firewall(nuFirewall,can2,gw,[allInputs],[allOutputs])

The above command will only show solutions (i.e., architectures) that all threats are mitigated by the suggested security
patterns. To also show solutions where where not all threats are mitigated, make sure the following line is commented
out in examples/headlamp/architecture-headlamp-not-mit-sec-1.dlv (needs manual change)

:- nmit([Z,V,P,T]) .

(2) Run the following command to show the consequences of deploying security patterns on safety

./dlv examples/headlamp/architecture-headlamp-not-ctl-saf-2.dlv safety/safety.dlv utils/utils.dlv -nofinitecheck tradeoff/tradeoff.dlv -filter=conflSaf2Sec -filter=ntol -filter=fl -filter=conflSec2Saf -filter=fl2Hz

The command (2) will show that there is one conflict caused by the deployment of a firewall. 

conflSec2Saf(firewall,fl([firewall,firewall,omission,cat]))}

This conflict, however, does not lead to any identified hazard as explained in the paper, i.e., there is NO "fl2Hz(firewall,hzHLSys)"

(3) Run the following command to show the consequences of deploying safety patterns on security

./dlv examples/headlamp/architecture-headlamp-not-mit-sec-2.dlv  security/security.dlv utils/utils.dlv -nofinitecheck tradeoff/tradeoff.dlv -filter=conflSec2Saf -filter=threat -filter=firewall -filter=mitby

The command (3) will show that there are three new threats caused by the deployment of a monitor actuator (safmon)

-> threat([safMon,[safMon,can2,gw,can3,obdConn],int,sev])
-> threat([safMon,[safMon,can2,gw,can1,bt],int,sev]) 
-> threat([safMon,[safMon,can2,gw,can1,cell],int,sev])

These threats, however, can be mitigated by the previously deployed firewall. One can see this by running the following command

./dlv examples/headlamp/architecture-headlamp-not-mit-sec-2.dlv  security/security.dlv utils/utils.dlv -nofinitecheck tradeoff/tradeoff.dlv -filter=mitby

As output, you will what security pattern mitigated which threat, including:

-> mitby([safMon,[safMon,can2,gw,can3,obdConn],int,sev],firewall) 
-> mitby([safMon,[safMon,can2,gw,can1,bt],int,sev],firewall) 
-> mitby([safMon,[safMon,can2,gw,can1,cell],int,sev],firewall)

Note: There is NO 'conflSec2Saf' because the new threats are mitigated by the preivously deployed firewall, as shown above.

-----------------------------------------------------------------
Instruction for running the BMS case study 
(NOT presented in the paper)
-----------------------------------------------------------------

(0) Run the following command to explore which safety patterns are recommended to tolerate the identified failures

./dlv examples/battery/architecture-battery-not-ctl-saf-1.dlv safety/safety.dlv utils/utils.dlv -filter=fl2Hz -filter=tolby -filter=fl -filter=ntol -filter=safMon -filter=tmr

The command (0) will provide 3 complete solutions that tolerate all identified failures.

(1) Run the following command to explore which security patterns are recommended to mitigate the identified threats

./dlv examples/battery/architecture-battery-not-mit-sec-1.dlv  safety/safety.dlv security/security.dlv utils/utils.dlv -nofinitecheck tradeoff/tradeoff.dlv -filter=threat -filter=firewall -filter=mitby -filter=secMonCP -filter=secMonCH

The command (1) will provide 5 complete solutions that mitigate all identified threats. Three of these solutions are shown below. We present solution c) in the paper.

(2) Run the following command to show the consequences of deploying security patterns on safety

./dlv examples/battery/architecture-battery-not-ctl-saf-2.dlv  safety/safety.dlv utils/utils.dlv tradeoff/tradeoff.dlv -filter=conflSec2Saf -filter=fl2Hz -filter=firewall

The command (2) will show that there is one conflict (i.e., failure) caused by the deployment of a firewall.  This failure will lead to an identified hazard, as shown below.

-> fl2Hz(firewall,hzBat)
-> conflSec2Saf(firewall,fl([firewall,firewall,omission,cat]))

This failure is tolerated by the deployment of a HDR automatically recommended by our machinery.

-> hdr(dmoon,bm,ci,moon1,moon2,voterdm,vtoutdm,bat)

(3) Run the following command to show the consequences of deploying safety patterns on security

./dlv examples/battery/architecture-battery-not-mit-sec-2.dlv  security/security.dlv utils/utils.dlv -nofinitecheck tradeoff/tradeoff.dlv -filter=conflSaf2Sec -filter=threat

The command (3) will show that there is one new threat caused by the deployment of a HDR

-> threat([voterdm,ci,int,sev])}

This threat is mitigated by the deployment of a security monitor (secMonCP) automtically recommended by our machinery.

-> secMonCP(nuSecMonCP,voterdm,[ic],[oc],dec,[min],[mout])

