# XMesh

XMesh is a mesh network protocol for communication via nodes powered by small solar cells.

Unfortunately I noticed afterwards that there is already a much older project called XMesh Protocol developed by memsic. Both projects are neither related nor comparable. IOTool's project here has an experimental status.

## solar challenge

I developed this protocol to be able to access remote sensors via repeaters. A repeater requires on average 1 mA current to be supplied with sufficient energy even in cloudy conditions or via a lamp. The advantage of solar cells is that the repeater does not require a power supply or batteries.

Low-cost nRF24 transceivers with an Atmel microcontroller can be used as repeaters, which are supplied with a 0.47F supercap via a small 5V solar cell. The complete hardware costs about three Euro. The system also works in winter because the super capacitor is less susceptible to cold environments than rechargeable batteries.

## network topology

XMesh is a fully meshed network. Each node communicates with every other node if it can send messages to or receive messages from this node. The mesh network can be connected to the Internet via a gateway.

The protocol was designed for nRF24 in the 2.4 GHz frequency range. Alternatively beacon frames from Wifi or Bluetooth can be used. XMesh can also be used in other frequency ranges or transmitted via infrared.

The mesh network can be controlled by a primary node and by some secondary nodes. There is no limit to the number of repeater nodes. 

## communication design

Unlike other systems, XMesh nodes are not permanently connected to each other. All messages are sent via broadcast without direct confirmation of receipt. The latency is extremely high and the data rate is very low.  The runtime of a data packet over 10 repeaters is about 20 seconds.

The reason for this is the target power consumption of 1 mA on average and the fact that nodes are not coupled.  Every few milliseconds the repeater wakes up and forwards the messages or receives new messages.

XMesh does not claim to be fast or reliable, but to enable extremely low cost nodes powered by solar cells.  With a larger solar cell or a permanent power supply the latency can be increased dynamically. At low energy levels, latency can be further increased and power consumption reduced.

### time slots

The time slots are aligned to the deep sleep of the Atmel microcontroller and to the transmit mode of nRF24. The protocol is optimized for continuous forwarding.  The reception time is selected so that at least one packet is received by a neighboring node.  The duration of the deep sleep is designed for a current consumption of 1.5 mA.

    loop (forever) {
      switch (energy) {
        case max: n=7;  // 2.56mA
        case mid: n=19; // 1.52mA
        case low: n=37; // 1.24mA
      }
      for (i=1..n) {
        for (f=1..2) {
          toogle_sendfreq;
          for (r=1..4) {
            senddata_40us;
            waitact_250us;
          }
        }
        deepsleep_32ms;
        wakeupboot_4ms;
        delay_1ms;
      }
      toogle_recvfreq;
      recvdata_39ms;
    }

This method enables data exchange between two neighboring nodes without time synchronization between each other. Each node inserts a random drift of 0..2 by varying the number of transmissions "n". This makes a permanent overlapping of reception times unlikely.

Two different frequencies are used for communication.  It is transmitted directly one after the other on both channels.  Reading takes place alternately on both channels.
