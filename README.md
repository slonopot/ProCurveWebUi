# Intro

Since Java in browsers is far gone, the only option to actually configure these switches today is telnet. However, I believe that if I bought 100% of a switch, I'd like to use a 100% of a switch, which is why I did this:

1. Got agent.jar from http://{switch}/classes/agent.jar
2. Unpacked it with JADX
3. Got these relevant classes: VLANmain.java DeviceStatus.java GenericBob.java InfinityBob.java MemberCandidateList.java PortGraph.java StackControl.java SwitchBob.java VLANmodifyPanel.java VLANprotocolPanel.java VLANTable.java StackConfig.java
4. Fed all this prehistoric garbage to Claude and asked it to make a new web control panel for the switch using the original logic from agent.jar
5. ???
6. Profit

The result is a 100% vibecoded piece of garbage that actually works. What's necessary is to disable CORS (for Firefox it's done by installing an extenstion called CORS Everywhere, for Chrome it's with --disable-web-security flag and there's also a requirement to supply a profile directory, google it). After that the page will actually connect to the switch and show stuff. You can also see the actual requests in the debugger and make your own requests (i.e. toggling ports on demand or record stats or whatever), some parsing will be needed because it's ancient but it's alright. There's also API to do some stuff in bulk from the browser console. Really sorry for the guys that didn't have modern tools back then, the agent.jar looks like a lot of effort which is now unusable.

# How to use this

1. In Firefox install [CORS Everywhere](https://addons.mozilla.org/en-US/firefox/addon/cors-everywhere/), enable it, go to `about:config` and set `security.mixed_content.block_active_content` to `false`. Run Chrome with `--disable-web-security --user-data-dir="./whatever"`, but I haven't tested it.
2. Open https://slonopot.github.io/ProCurveWebUi/
3. Input IP of the switch, the interface should work. Ports are allowed if you're dst-nat forwarding.

I've only tested VLANs, stats and port management, stack management is untested. Remember that this is garbage and shouldn't be trusted. Strictly for educational purposes.
