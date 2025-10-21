<h1>Self-Hosted NVR with Frigate on Debian</h1>
<h2>Description</h2>
For this homelab project, I repurposed an existing desktop and deployed Debian Linux as the base operating system, then configured Frigate as a dedicated NVR using Docker Compose. A Cisco PoE switch is used to supply both power and network connectivity to the IP cameras over a single Ethernet run, eliminating the need for external power supplies and simplifying the overall infrastructure.
<h2>Hardware Used</h2>
<ul>
  <li>Repurposed desktop running Debian 13 as the host system</li>
  <li>IP camera with RTSP streaming capability</li>
  <li>Cisco PoE switch providing both power and network connectivity over a single Ethernet cable</li>
  <li>Dedicated storage for video recordings and retention</li>
</ul>
<h2>Camera Requirements and Configuration</h2>
<p>
For Frigate to operate properly, you will need an IP camera capable of streaming via RTSP. In my setup, I am using the
<a href="https://www.amazon.com/dp/B083G9KT4C?amp=">Amcrest IP5M-T1179EW</a>. Additional recommended cameras and supported hardware can be found in the
<a href="https://docs.frigate.video/frigate/hardware">official Frigate documentation</a>. The camera is connected through a Cisco PoE switch, which provides both power and network connectivity over a single Ethernet cable, simplifying the installation and wiring process.
</p>
<b>Camera Configuration</b>
<ul>
  <li>Test the camera prior to mounting to verify proper operation</li>
  <li>Configure a static IP address and create dedicated credentials for Frigate access</li>
  <li>Mount the camera in the desired location and run the Ethernet cable to the PoE switch</li>
</ul>
<h2>Debian Setup</h2>
<ul>
  <li>Creating a Bootable USB</li>
  <ul>
    <li>Download the latest Debian ISO from the official website</li>
    <li>Use a tool such as <i>Rufus</i> to flash the ISO onto a USB drive and make it bootable</li>
  </ul>
  <li>Installing Debian</li>
  <ul>
    <li>Insert the USB drive into the target machine and select it as the boot device from the BIOS/UEFI</li>
    <li>Proceed through the installation using the default options; I opted not to install a desktop environment since the system will be administered remotely via SSH</li>
  </ul>
  <li>Installing Docker Engine</li>
  <ul>
    <li>Install Docker by following the official instructions from the
      <a href="https://docs.docker.com/engine/install/debian/">Docker documentation</a> using the APT repository method</li>
  </ul>
</ul>
<h2>Getting Frigate Installed</h2>
<p>
For additional details about Frigate and its capabilities, you can refer to the official 
<a href="https://docs.frigate.video/guides/getting_started/">documentation</a>.
</p>
<b>Installation</b>
<ul>
  <li>On the Debian server, I created a dedicated <code>Frigate</code> directory to store all configuration and service files. Inside this directory, I added a <code>config</code> folder and a <code>docker-compose.yaml</code> file. For video storage, I am using a separate drive to keep all camera footage. I mounted this drive to <code>/mnt</code> and created a directory named <code>frigate-videos</code> to serve as the storage location.</li>

  <pre>## Creating project directory and base files
mkdir Frigate
cd Frigate
mkdir config
touch docker-compose.yaml</pre>

  <pre>## Mounting the external drive and preparing the storage directory
sudo mount /dev/sda /mnt/frigate-videos
cd /mnt/frigate-videos
mkdir storage</pre>

  <li>Next, configure the <code>docker-compose.yaml</code> file according to your setup. 
You can view my full configuration example on 
<a href="https://github.com/evertorres0/Frigate/blob/main/config.yml" target="_blank">GitHub</a>. 
Depending on your environment, you may need to adjust camera streams, storage paths, or network settings. 
For further guidance, refer to the official 
<a href="https://docs.frigate.video/frigate/installation/">Frigate documentation</a>.</li>

  <li>Once the compose file is ready, start the service using <code>docker compose up -d</code>. During initial startup, Frigate will create a admin user and password which can be see in the starup or by runing <code>docker log frigate</code>. You can sign in using the automatically created admin account by navigating to <code>https://&lt;server-ip&gt;:8971</code> in your browser.</li>
</ul>
<h2>Frigate Configuration</h2>
<p>
Once Frigate is running, the configuration file can be accessed directly from the <code>config</code> directory created during setup, or edited through the web interface for convenient in-browser modifications.
</p>
<p>
Frigate is configured using a <code>config.yml</code> file which defines your camera streams, storage paths, and recording behavior. 
For reference, I have published my full configuration file on 
<a href="https://github.com/evertorres0/Frigate/blob/main/docker-compose.yaml" target="_blank">GitHub</a>, which can be used as a template or starting point for your own setup. 
You may customize the camera name, RTSP URL, retention policy, and storage location depending on your environment and requirements.
</p>
<p>
The configuration below disables MQTT since this setup is focused on continuous recording rather than event-based alerts. 
Login rate limiting is enabled through the <code>auth</code> section for added security. 
Recordings are retained for 5 days, with Frigate performing cleanup every 60 minutes. 
The <code>go2rtc</code> block defines both the main RTSP stream and a lower-resolution substream for the same camera.
Although object detection is currently disabled, the configuration is prepared for future use by assigning the substream to the <code>detect</code> role. 
The primary stream is used solely for high-quality recording, and hardware acceleration is enabled via VAAPI for improved performance on Intel-based systems.
</p>
<p>
For more advanced options — including motion detection, object detection, hardware acceleration, MQTT integration, and multi-camera setups — refer to the official 
<a href="https://docs.frigate.video/configuration/reference">Frigate configuration documentation</a>.
</p>
