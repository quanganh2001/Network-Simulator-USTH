Bus network topology:

1. Building a bus network topology of 3 nodes (0, 1, 2) on a LAN using CSMA Channel.
2. Implement scenario of node 0 is a client and node 2 is an echo server. They exchange 100 packets within 10s. Capture the pcap trace from node 1.
3. Create a topology of “n” CSMA devices. Implement the scenarios of n=5, on which there are 2 pairs of client-server (0-2) and (1-3) exchanging packets at the same time. Each pair exchanges 10 packets/s. Run the simulation for 10s.
    
    Compare packet delivery ratio and average delay of received packets at each server.
    

Source code:

```cpp
#include "ns3/core-module.h"
#include "ns3/network-module.h"
#include "ns3/csma-module.h"
#include "ns3/internet-module.h"
#include "ns3/applications-module.h"
#include "ns3/ipv4-global-routing-helper.h"

using namespace ns3;

NS_LOG_COMPONENT_DEFINE ("Bus network topology");

int 
main (int argc, char *argv[])
{
  bool verbose = true;
  uint32_t nCsma = 5;

  CommandLine cmd (__FILE__);
  cmd.AddValue ("nCsma", "Number of \"extra\" CSMA nodes/devices", nCsma);
  cmd.AddValue ("verbose", "Tell echo applications to log if true", verbose);

  cmd.Parse (argc,argv);

  if (verbose)
    {
      LogComponentEnable ("UdpEchoClientApplication", LOG_LEVEL_INFO);
      LogComponentEnable ("UdpEchoServerApplication", LOG_LEVEL_INFO);
    }

  nCsma = nCsma == 0 ? 1 : nCsma;

  NodeContainer csmaNodes;
  csmaNodes.Create (nCsma);

  CsmaHelper csma;
  csma.SetChannelAttribute ("DataRate", StringValue ("100Mbps"));
  csma.SetChannelAttribute ("Delay", TimeValue (NanoSeconds (6560)));

  NetDeviceContainer csmaDevices;
  csmaDevices = csma.Install (csmaNodes);

  InternetStackHelper stack;
  stack.Install (csmaNodes);

  Ipv4AddressHelper address;
  address.SetBase ("10.1.2.0", "255.255.255.0");
  Ipv4InterfaceContainer csmaInterfaces;
  csmaInterfaces = address.Assign (csmaDevices);

  UdpEchoServerHelper echoServer (9);

  ApplicationContainer serverApps1 = echoServer.Install (csmaNodes.Get (2));
  serverApps1.Start (Seconds (1.0));
  serverApps1.Stop (Seconds (12.0));

  UdpEchoClientHelper echoClient1 (csmaInterfaces.GetAddress (2), 9);
  echoClient1.SetAttribute ("MaxPackets", UintegerValue (10));
  echoClient1.SetAttribute ("Interval", TimeValue (Seconds (1)));
  echoClient1.SetAttribute ("PacketSize", UintegerValue (1024));

  ApplicationContainer clientApps1 = echoClient1.Install (csmaNodes.Get (0));
  clientApps1.Start (Seconds (2.0));
  clientApps1.Stop (Seconds (12.0));

  ApplicationContainer serverApps2 = echoServer.Install (csmaNodes.Get (3));
  serverApps2.Start (Seconds (1.0));
  serverApps2.Stop (Seconds (12.0));

  UdpEchoClientHelper echoClient2 (csmaInterfaces.GetAddress (3), 9);
  echoClient2.SetAttribute ("MaxPackets", UintegerValue (10));
  echoClient2.SetAttribute ("Interval", TimeValue (Seconds (1)));
  echoClient2.SetAttribute ("PacketSize", UintegerValue (1024));

  ApplicationContainer clientApps2 = echoClient2.Install (csmaNodes.Get (1));
  clientApps2.Start (Seconds (2.0));
  clientApps2.Stop (Seconds (12.0));

  Ipv4GlobalRoutingHelper::PopulateRoutingTables ();

  csma.EnablePcap ("second", csmaDevices.Get (1), true);

  Simulator::Run ();
  Simulator::Destroy ();
  return 0;
}
```

**Result:**

```
At time +2s client sent 1024 bytes to 10.1.2.3 port 9
At time +2s client sent 1024 bytes to 10.1.2.4 port 9
At time +2.00012s server received 1024 bytes from 10.1.2.2 port 49153
At time +2.00012s server sent 1024 bytes to 10.1.2.2 port 49153
At time +2.00324s client received 1024 bytes from 10.1.2.4 port 9
At time +2.00612s server received 1024 bytes from 10.1.2.1 port 49153
At time +2.00612s server sent 1024 bytes to 10.1.2.1 port 49153
At time +2.01224s client received 1024 bytes from 10.1.2.3 port 9
At time +3s client sent 1024 bytes to 10.1.2.3 port 9
At time +3s client sent 1024 bytes to 10.1.2.4 port 9
At time +3.00009s server received 1024 bytes from 10.1.2.1 port 49153
At time +3.00009s server sent 1024 bytes to 10.1.2.1 port 49153
At time +3.00019s client received 1024 bytes from 10.1.2.3 port 9
At time +3.00065s server received 1024 bytes from 10.1.2.2 port 49153
At time +3.00065s server sent 1024 bytes to 10.1.2.2 port 49153
At time +3.00074s client received 1024 bytes from 10.1.2.4 port 9
At time +4s client sent 1024 bytes to 10.1.2.3 port 9
At time +4s client sent 1024 bytes to 10.1.2.4 port 9
At time +4.00009s server received 1024 bytes from 10.1.2.1 port 49153
At time +4.00009s server sent 1024 bytes to 10.1.2.1 port 49153
At time +4.00019s client received 1024 bytes from 10.1.2.3 port 9
At time +4.00029s server received 1024 bytes from 10.1.2.2 port 49153
At time +4.00029s server sent 1024 bytes to 10.1.2.2 port 49153
At time +4.00038s client received 1024 bytes from 10.1.2.4 port 9
At time +5s client sent 1024 bytes to 10.1.2.3 port 9
At time +5s client sent 1024 bytes to 10.1.2.4 port 9
At time +5.00009s server received 1024 bytes from 10.1.2.1 port 49153
At time +5.00009s server sent 1024 bytes to 10.1.2.1 port 49153
At time +5.00019s client received 1024 bytes from 10.1.2.3 port 9
At time +5.00033s server received 1024 bytes from 10.1.2.2 port 49153
At time +5.00033s server sent 1024 bytes to 10.1.2.2 port 49153
At time +5.00042s client received 1024 bytes from 10.1.2.4 port 9
At time +6s client sent 1024 bytes to 10.1.2.3 port 9
At time +6s client sent 1024 bytes to 10.1.2.4 port 9
At time +6.00009s server received 1024 bytes from 10.1.2.1 port 49153
At time +6.00009s server sent 1024 bytes to 10.1.2.1 port 49153
At time +6.00019s client received 1024 bytes from 10.1.2.3 port 9
At time +6.00046s server received 1024 bytes from 10.1.2.2 port 49153
At time +6.00046s server sent 1024 bytes to 10.1.2.2 port 49153
At time +6.00055s client received 1024 bytes from 10.1.2.4 port 9
At time +7s client sent 1024 bytes to 10.1.2.3 port 9
At time +7s client sent 1024 bytes to 10.1.2.4 port 9
At time +7.00009s server received 1024 bytes from 10.1.2.1 port 49153
At time +7.00009s server sent 1024 bytes to 10.1.2.1 port 49153
At time +7.00019s client received 1024 bytes from 10.1.2.3 port 9
At time +7.00039s server received 1024 bytes from 10.1.2.2 port 49153
At time +7.00039s server sent 1024 bytes to 10.1.2.2 port 49153
At time +7.00049s client received 1024 bytes from 10.1.2.4 port 9
At time +8s client sent 1024 bytes to 10.1.2.3 port 9
At time +8s client sent 1024 bytes to 10.1.2.4 port 9
At time +8.00009s server received 1024 bytes from 10.1.2.1 port 49153
At time +8.00009s server sent 1024 bytes to 10.1.2.1 port 49153
At time +8.00019s client received 1024 bytes from 10.1.2.3 port 9
At time +8.00044s server received 1024 bytes from 10.1.2.2 port 49153
At time +8.00044s server sent 1024 bytes to 10.1.2.2 port 49153
At time +8.00053s client received 1024 bytes from 10.1.2.4 port 9
At time +9s client sent 1024 bytes to 10.1.2.3 port 9
At time +9s client sent 1024 bytes to 10.1.2.4 port 9
At time +9.00009s server received 1024 bytes from 10.1.2.1 port 49153
At time +9.00009s server sent 1024 bytes to 10.1.2.1 port 49153
At time +9.00019s client received 1024 bytes from 10.1.2.3 port 9
At time +9.00065s server received 1024 bytes from 10.1.2.2 port 49153
At time +9.00065s server sent 1024 bytes to 10.1.2.2 port 49153
At time +9.00075s client received 1024 bytes from 10.1.2.4 port 9
At time +10s client sent 1024 bytes to 10.1.2.3 port 9
At time +10s client sent 1024 bytes to 10.1.2.4 port 9
At time +10.0001s server received 1024 bytes from 10.1.2.1 port 49153
At time +10.0001s server sent 1024 bytes to 10.1.2.1 port 49153
At time +10.0002s client received 1024 bytes from 10.1.2.3 port 9
At time +10.0003s server received 1024 bytes from 10.1.2.2 port 49153
At time +10.0003s server sent 1024 bytes to 10.1.2.2 port 49153
At time +10.0004s client received 1024 bytes from 10.1.2.4 port 9
At time +11s client sent 1024 bytes to 10.1.2.3 port 9
At time +11s client sent 1024 bytes to 10.1.2.4 port 9
At time +11.0001s server received 1024 bytes from 10.1.2.1 port 49153
At time +11.0001s server sent 1024 bytes to 10.1.2.1 port 49153
At time +11.0002s client received 1024 bytes from 10.1.2.3 port 9
At time +11.0003s server received 1024 bytes from 10.1.2.2 port 49153
At time +11.0003s server sent 1024 bytes to 10.1.2.2 port 49153
At time +11.0004s client received 1024 bytes from 10.1.2.4 port 9
```

**Conclusion**

- The packet delivery ratio of received packets at the 2 servers is the same.
- The average delay of received packets at server 2 is larger than at server 1.
