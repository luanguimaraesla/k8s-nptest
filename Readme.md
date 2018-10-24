Based on: [kubernetes/perf-tests](https://github.com/kubernetes/perf-tests/blob/master/network/benchmarks/netperf)

Docker Image: [here](https://hub.docker.com/r/luanguimaraesla/netperf/)

# Benchmarking Kubernetes Cluster-Mesh Networking Performance

## Objectives:
A standardized benchmark to measure Kubernetes networking performance on Kubernetes platforms configured with cluster-mesh.

## Implementation and Test Methodology

The benchmark can be executed via a single Go binary invocation that triggers all the automated testing located in the orchestrator and worker pods as seen below. The test uses a custom docker container that has the go binary and iperf3 and other tools built into it. 
The orchestrator pod coordinates the worker pods to run tests in serial order for the 6 scenarios described below, at MTUs (MSS tuning for TCP and direct packet size tuning for UDP). The MTU range covers 96 till 1422 (AWS VPC Peering MTU limits) in steps of 64.

Using node labels, the Worker Pods 1 and 2 are placed on the same Kubernetes node, Worker Pod 3 is placed on a different node in the same cluster and the Worker 4 is placed on a node of the other cluster. The workers all communicate with the orchestrator pod service using simple golang rpcs and request work items. A minimum of two Kubernetes worker nodes on cluster A are necessary for this test, and one for cluster B.

The 5 major network traffic paths are combination of Pod IP vs Virtual IP and whether the pods are co-located on the same node/VM versus a remotely located pod.

* Same VM using Pod IP: Same VM Pod to Pod traffic tests from Worker 1 to Worker 2 using its Pod IP.

* Same VM using Cluster/Virtual IP: Same VM Pod to Pod traffic tests from Worker 1 to Worker 2 using its Service IP (also known as its Cluster IP or Virtual IP).

* Remote VM using Pod IP: Worker 3 to Worker 2 traffic tests using Worker 2 Pod IP.

* Remote VM using Cluster/Virtual IP: Worker 3 to Worker 2 traffic tests using Worker 2 Cluster/Virtual IP.

* Same VM Pod Hairpin: Worker 2 to itself using Cluster IP.

* Remote Cluster using Pod IP: Worker 1 to Worker 4 traffic tests using Worker 4 Pod IP.

The orchestrator and worker pods run independently of the initiator script, with the orchestrator pod sending work items to workers till the testcase schedule is complete. For worker 4 you need to update the `manifests/clusterB/netperf-w4-rc.yml` and put the IP of the orchestrator node from the other cluster before applying it.
The iperf output (both TPC and UDP modes) and the netperf TCP output from all worker nodes is uploaded to the orchestrator pod where it is filtered and the results are written to the output file as well as to stdout log.
Default file locations are /tmp/result.csv and /tmp/output.txt for the raw results.

## Output Raw CSV data [OUTDATED]
**All units in the csv file are in Gbits/second**
```console
ALL TESTCASES AND MSS RANGES COMPLETE - GENERATING CSV OUTPUT
the output for each MSS testpoint is a single value in Mbits/sec
MSS                                          ; Maximum; 96; 160; 224; 288; 352; 416; 480; 544; 608; 672; 736; 800; 864; 928; 992; 1056; 1120; 1184; 1248; 1312; 1376;
1 iperf TCP. Same VM using Pod IP            ;49167.000000;43366;43046;43695;44122;43698;44398;43318;43853;48592;44284;48760;42945;48154;48453;48991;48607;42706;49167;43910;48531;43626;
2 iperf TCP. Same VM using Virtual IP        ;47440.000000;41262;42580;41333;42267;42104;43004;43345;43286;47022;41568;47025;42729;47412;46839;47440;47172;41869;47278;42731;46945;42720;
3 iperf TCP. Remote VM using Pod IP          ;9274.000000;741;1352;2066;2521;2957;3481;4192;4414;5066;5638;6092;6619;6795;7432;8265;8385;8708;8503;9181;9234;9274;
4 iperf TCP. Remote VM using Virtual IP      ;9265.000000;682;1349;1904;2369;2923;3628;4025;4260;4931;5760;6086;6799;6948;6819;8374;8686;8930;9125;9169;9237;9265;
5 iperf TCP. Remote Cluster using Pod IP     ;1633.000000;327;366;518;848;863;1189;796;856;833;1077;656;619;971;968;630;1257;1633;1437;1297;1048;1494;
6 iperf TCP. Hairpin Pod to own Virtual IP   ;48187.000000;41122;41975;41136;41803;41789;41781;42512;40908;47353;42854;46948;42710;46630;46688;46534;46942;41325;48187;42339;47202;43041;
7 iperf UDP. Same VM using Pod IP            ;2399.000000;2399;
8 iperf UDP. Same VM using Virtual IP        ;2330.000000;2330;
9 iperf UDP. Remote VM using Pod IP          ;2226.000000;2226;
10 iperf UDP. Remote VM using Virtual IP     ;2157.000000;2157;
11 iperf UDP. Remote Cluster using Pod IP    ;2224.000000;2224;
12 netperf. Same VM using Pod IP             ;25921.830000;25921.83;
13 netperf. Same VM using Virtual IP         ;0.000000;0.00;
14 netperf. Remote VM using Pod IP           ;4921.630000;4921.63;
15 netperf. Remote VM using Virtual IP       ;0.000000;0.00;
16 netperf. Remote Cluster using Pod IP      ;0.000000;0.00;
END CSV DATA
```
