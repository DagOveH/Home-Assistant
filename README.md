# Home-Assistant
Playing with Home Assistant

Installing Home Assistant 4.6 OVA on VMWare ESXi 6.7.0 (Build 8169922)

Download https://github.com/home-assistant/operating-system/releases/download/4.6/hassos_ova-4.6.ova

Extract with tar -xvf hass_ova-4.6.ova

Modify this section of the file  home-assistant.ovf to

4.6
      <Item>
        <rasd:AutomaticAllocation>true</rasd:AutomaticAllocation>
        <rasd:Connection>Bridged</rasd:Connection>
        <rasd:Description>Ethernet adapter</rasd:Description>
        <rasd:ElementName>eth0</rasd:ElementName>
        <rasd:InstanceID>6</rasd:InstanceID>
        <rasd:ResourceSubType>E1000</rasd:ResourceSubType>
        <rasd:ResourceType>10</rasd:ResourceType>
      </Item>

Create a new ova file with the command: tar -cvf ../hassos_ova-4.6_new.ova *

Use the Create VM and Deploy a virtual machine from an OVF or OVA file
choose the newly created ova file
