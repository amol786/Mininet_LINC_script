<b>Goal</b>: Seting up LINC-OE simple optical topology controlled by iControl and a packet simple topology controlled by POX. 

<b>Requirements:</b>
A basic knowlege of LINC-OE, TAP interfaces, POX, Erlang language and linux CLI is required. 
Doing https://github.com/Ehsan70/Mininet_LINC_script/blob/master/LINCoe_and_iControl.md tutorial is a must. 

<b>Environment: <b> I have used the VM from sdn hub, I recommond you do the same. Link for installation is provided below: http://sdnhub.org/tutorials/sdn-tutorial-vm/

<b>Road Map: </b>This document has two sections for setup: 

 1. setting up the optical network   
 2. setting up the packet network
 
# 1. setting up the optical network #
 a. Run iControl: 
 ```shell
 cd loom/iControl
 rel/icontrol/bin/icontrol console
 ```
 b. Clearting the tap interfaces: 
 ```shell
 sudo tunctl -t tap0
 sudo tunctl -t tap1
 sudo ip link set dev tap0 up
 sudo ip link set dev tap1 up
 ```
 c. Set up the `sys.config` file: 
 `rel/files/sys.config` file for the network shown above should looks as following:
 ```erlang
 [{linc,
   [{of_config,disabled},
    {capable_switch_ports,
     [{port,1,[{interface,"tap0"}]},
      {port,2,[{interface,"dummy"}, {type, optical}]},
      {port,3,[{interface,"dummy"}, {type, optical}]},
      {port,4,[{interface,"dummy"}, {type, optical}]},
      {port,5,[{interface,"dummy"}, {type, optical}]},
      {port,6,[{interface,"tap1"}]}
     ]},
    {capable_switch_queues, []},
    {optical_links, [{{1,2}, {2,1}}, {{2,2},{3,1}}]},
    {logical_switches,
     [{switch,1,
       [{backend,linc_us4_oe},
        {controllers,[{"Switch0-Controller","localhost",6653,tcp}]},
        {controllers_listener,disabled},
        {queues_status,disabled},
        {datapath_id, "00:00:00:00:00:01:00:01"},
        {ports,[{port,1,[{queues,[]}, {port_no, 1}]},
                {port,2,[{queues,[]}, {port_no, 2}]}
               ]}]},
      {switch,2,
       [{backend,linc_us4_oe},
        {controllers,[{"Switch0-Controller","localhost",6653,tcp}]},
        {controllers_listener,disabled},
        {queues_status,disabled},
        {datapath_id, "00:00:00:00:00:01:00:02"},
        {ports,[{port,3,[{queues,[]}, {port_no, 1}]},
                {port,4,[{queues,[]}, {port_no, 2}]}
               ]}]},
      {switch,3,
       [{backend,linc_us4_oe},
        {controllers,[{"Switch0-Controller","localhost",6653,tcp}]},
        {controllers_listener,disabled},
        {queues_status,disabled},
        {datapath_id, "00:00:00:00:00:01:00:03"},
        {ports,[{port,5,[{queues,[]}, {port_no, 1}]},
                {port,6,[{queues,[]}, {port_no, 2}]}
               ]}]}
     ]}]},
  {of_protocol, [{no_multipart, false}]},
  {enetconf,
   [{capabilities,[{base,{1,1}},{startup,{1,0}},{'writable-running',{1,0}}]},
    {callback_module,linc_ofconfig},
    {sshd_ip,any},
    {sshd_port,1830},
    {sshd_user_passwords,[{"linc","linc"}]}]},
  {epcap,
   [{verbose, false},
    {stats_interval, 10},
    {buffer_size, 73400320}]},
  {lager,
   [{handlers,
     [{lager_console_backend,debug},
      {lager_file_backend,
       [{"log/error.log",error,10485760,"$D0",5},
        {"log/debug.log",debug,10485760,"$D0",5},
        {"log/console.log",info,10485760,"$D0",5}]}]}]},
  {sasl,
   [{sasl_error_logger,{file,"log/sasl-error.log"}},
    {errlog_type,error},
    {error_logger_mf_dir,"log/sasl"},
    {error_logger_mf_maxbytes,1048576000000},
    {error_logger_mf_maxfiles,5}]},
  {sync,
   [{excluded_modules, [procket]}]}].
 ```
 d. start LINC-OE: 
 ```shell
 make rel && sudo rel/linc/bin/linc console
 ```
 e. Add some flows to the optical switches. 
 Type the following commands in the iControl window:
 ```erlang
 iof:oe_flow_tw(2,100,1,2,20).
 iof:oe_flow_ww(1,100,1,20,2,20).
 iof:oe_flow_wt(3,100,1,20,2).
 
 iof:oe_flow_wt(2,100,2,20,1).
 iof:oe_flow_ww(1,100,2,20,1,20).
 iof:oe_flow_tw(3,100,2,1,20).
 ```
 > Note that you need to add flows for both directions. 
 
 So now if put stuff (packets) in one of the tap interfaces (using tcpreply) it will apear on the tap. 
 
 
 
 
 